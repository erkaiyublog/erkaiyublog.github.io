---
published: true
title: Source Code Study - QEMU cpu_exec.c
tags: qemu SCS
---

Lately, I've been trying to record assembly execution trace of QEMU emulations. To grasp a better understanding of this process, I decided to take a look at the source code of QEMU, ```accel/tcg/cpu_exec.c``` in particular. As I'm reading the source code, I write this blog post as a review of what I've learnt from it. 

# Table of Contents
* TOC
{:toc}

# Background Knowlege 
QEMU’s TCG (Tiny Code Generator) dynamically translates guest CPU instructions into host CPU instructions in small chunks called Translation Blocks (TBs). ```cpu_exec.c``` is the file containing functions related to TB executions. 

The QEMU version I studied is the [source code for the stable release 9.2.2](https://github.com/qemu/qemu/tree/ea35a5082a5fe81ce8fd184b0e163cd7b08b7ff7) (released on Feb 24th 2025). The file I'm focusing on is located at [accel/tcg/cpu_exec.c](https://github.com/qemu/qemu/blob/ea35a5082a5fe81ce8fd184b0e163cd7b08b7ff7/accel/tcg/cpu-exec.c).  

# Critical Functions in ```cpu_exec.c```
## ```cpu_tb_exec```
This function executes a given TB, and fix up the CPU state afterwards if necessary. A macro ```QEMU_DISABLE_CFI``` is applied to the function header, to disable CFI (Control Flow Integrity) since QEMU executes TB by making **indirect** function calls.

If TB execution logging is enabled (you can do this by setting ```-d exec -D qemu.log``` in your QEMU run command), ```log_cpu_exec``` function will be called here. 

The critical section comes as follows,

    ret = tcg_qemu_tb_exec(cpu_env(cpu), tb_ptr);
    cpu->neg.can_do_io = true;
    qemu_plugin_disable_mem_helpers(cpu);

The ```tcg_qemu_tb_exec``` function executes the TB, stores the address of the next TB in ```ret```. After executing TB, I/O permission is restored. ```qemu_plugin_disable_mem_helpers(cpu)``` ensures that memory access helpers from plugins are disabled after execution.

After the critical section, ```last_tb``` is extracted from ```ret```, and the ```pc``` should be reset if this last TB is not executed completely.

    last_tb = tcg_splitwx_to_rw((void *)(ret & ~TB_EXIT_MASK));
    *tb_exit = ret & TB_EXIT_MASK;

    trace_exec_tb_exit(last_tb, *tb_exit);

    if (*tb_exit > TB_EXIT_IDX1) {
        /* We didn't start executing this TB (eg because the instruction
         * counter hit zero); we must restore the guest PC to the address
         * of the start of the TB.
         */
        
        /* ... */ 
    }

Note in the code snippet above, the function name ```trace_exec_tb_exit``` doesn't appear anywhere in the QEMU source folder, this is because the prefix ```trace_``` is probably applied during compilation, and there's a file containing the definition for ```exec_tb_exit``` at [accel/tcg/trace_events](https://github.com/qemu/qemu/blob/ea35a5082a5fe81ce8fd184b0e163cd7b08b7ff7/accel/tcg/trace-events).

After checking the completeness of the last TB execution, some GDB related stuff is envolved, and finally ```return last_tb;```

## ```cpu_handle_interrupt```
This function is being called over and over again in the "main loop" of cpu execution (function ```cpu_exec_loop```), it checks and handles potential interrupts during emulation.

At the beginning, it checks if the ```CF_NOIRQ``` flag is set,

    /*
     * If we have requested custom cflags with CF_NOIRQ we should
     * skip checking here. Any pending interrupts will get picked up
     * by the next TB we execute under normal cflags.
     */
    if (cpu->cflags_next_tb != -1 && cpu->cflags_next_tb & CF_NOIRQ) {
        return false;
    }

Then there's a super long if-else chain for different settings such as ```TARGET_i386```, ```replay_has_interrupt```, ```CPU_INTERRUPT_DEBUG```. But essentially, it calls ```tcg_ops->cpu_exec_interrupt``` to actually handle the interrupt, after handling the interrupt, ```last_tb``` will be set to ```NULL``` to indicate incomplete TB execution.

Finally, return ```true``` if the ```cpu_exec_loop``` should end (might because of no cpu instruction left).

## ```cpu_exec_loop```
The main execution loop.

It wraps everything in two nested while loops:

    /* if an exception is pending, we execute it here */
    while (!cpu_handle_exception(cpu, &ret)) {
        TranslationBlock *last_tb = NULL;
        int tb_exit = 0;z

        while (!cpu_handle_interrupt(cpu, &last_tb)) {
            /* ... */
        }
    }

Inside the inner while loop, the function runs with the following set of variables

    TranslationBlock *last_tb = NULL;
    int tb_exit = 0;
    TranslationBlock *tb;
    vaddr pc;         
    uint64_t cs_base; 
    uint32_t flags;   
    uint32_t cflags;

```pc, cs_base, flags``` are set by calling cpu_get_tb_cpu_state. ```cflags``` is set as 

    /*
        * When requested, use an exact setting for cflags for the next
        * execution.  This is used for icount, precise smc, and stop-
        * after-access watchpoints.  Since this request should never
        * have CF_INVALID set, -1 is a convenient invalid value that
        * does not require tcg headers for cpu_common_reset.
        */
    cflags = cpu->cflags_next_tb;
    if (cflags == -1) {
        cflags = curr_cflags(cpu);
    } else {
        cpu->cflags_next_tb = -1;
    }    

It appears to me that ```cflags``` will normally be set to ```cpu->cflags_next_tb``` and the special case is to set it to ```curr_cflags(cpu)```.

Next, find the next TB and add it to the cache. 

    tb = tb_lookup(cpu, pc, cs_base, flags, cflags);
    if (tb == NULL) {
        CPUJumpCache *jc;
        uint32_t h;

        mmap_lock();
        tb = tb_gen_code(cpu, pc, cs_base, flags, cflags);
        mmap_unlock();

        /*
        * We add the TB in the virtual pc hash table
        * for the fast lookup
        */
        h = tb_jmp_cache_hash_func(pc);
        jc = cpu->tb_jmp_cache;
        jc->array[h].pc = pc;
        qatomic_set(&jc->array[h].tb, tb);
    }

At the end of the inner while loop, make call to the ```cpu_loop_exec_tb``` to execute the current TB, 

    /* See if we can patch the calling TB. */
    if (last_tb) {
        tb_add_jump(last_tb, tb_exit, tb);
    }

    cpu_loop_exec_tb(cpu, tb, pc, &last_tb, &tb_exit);

Note that as indicated by the comment above, the function first checks if the calling TB can be patched by setting current ```tb``` as the jump destination of ```last_tb```.

# Other Functions in ```cpu_exec.c```
Here's an overview of all the functions included in the file. I highlighted the ones I found critical, and provided a brief description for the rest of them.

* align_clocks
    * Make use of a ```SyncClocks``` struct to align clock (by checking the difference between the current and the last ```cpu_icount```).
* print_delay
    * Call ```qemu_printf``` to print the difference in clock.
* init_delay_params 
    * Config ```SyncClocks``` for later clock alignment.
* tcg_cflags_has
    * One line, ```return cpu->tcg_cflags & flags;```
* tcg_cflags_set
    * One line, ```cpu->tcg_cflags |= flags;```
* curr_cflags
    * Return the current ```cpu->tcg_cflags```.
* tb_lookup_cmp
    * Compare TB (used by hash tables).
* tb_htable_lookup
    * Lookup TB in hash table by calling ```qht_lookup_custom```. 
* tb_lookup
    * Use input virtual address ```pc``` to lookup for TB in ```cpu->tb_jmp_cache->array```, when checking, the TB is identified by not only ```pc```, but also ```cs_base, flags, cflags```. 
    * If the TB is not in cache, call ```tb_htable_lookup``` to get it.
* log_cpu_exec
    * Log the execution of a TB. The overhead is pretty high in practice.
* check_for_breakpoints_slow
    * Perform a detailed check to determine if the ```pc``` corresponds to a defined breakpoint.
* check_for_breakpoints
    * Check if any checkpoint exists, if so, call ```check_for_breakpoints_slow```.
* lookup_tb_ptr
    * Look for an existing TB matching the current cpu state (call ```tb_lookup```). If found, return the code pointer.  If not found, return ```tcg_code_gen_epilogue``` so the program will return into ```cpu_tb_exec```.
* **cpu_tb_exec**
* cpu_exec_enter
    * Calls a ```cpu_exec_enter``` function if it is defined for the given ```cpu```. This function provides a hook for CPU architectures that need to perform certain pre-execution steps.
* cpu_exec_exit
    * Calls a ```cpu_exec_exit``` function if it is defined for the given ```cpu```. This function provides a hook for CPU architectures that need to perform certain post-execution steps.
* cpu_exec_longjmp_cleanup
    * Cleanup function for ```longjmp```.
* cpu_exec_step_atomic
    * This function executes a single instruction in an atomic context. 
* tb_set_jmp_target
    * Set the jump target for a given TB, by modifying ```tb->jmp_target_addr[n]``` to be the ```addr``` passed in.
* tb_add_jump
    * Links two given TBs ```tb``` and ```tb_next``` by calling ```tb_set_jmp_target```.
* cpu_handle_halt
    * Check if cpu is halted and executes a handler for the halt condition if necessary.
* cpu_handle_debug_exception
    * Handle debug exceptions by calling pre-defined ```tcg_ops->debug_excp_handler``` function.
* cpu_handle_exception
    * Handles exceptions, some ```replay_exception``` feature related with debugging is also handled by this function. 
* icount_exit_request
    * A boolean function indicting whether the CPU should stop execution based on the instruction count.
* **cpu_handle_interrupt**
* cpu_loop_exec_tb
    * This function is called in the main loop (in ```cpu_exec_loop```) to execute a chained TB. It executes TB by calling ```cpu_tb_exec``` function.
* **cpu_exec_loop**
* cpu_exec_setjmp
    * It first prepares setjmp context for exception handling, then call the ```cpu_exec_loop``` to start the main loop.
* cpu_exec
    * The "entry point" for CPU execution. This function starts all the clock alignment and cpu execution prologue, then call ```cpu_exec_setjmp``` which will call ```cpu_exec_loop``` to start the CPU execution loop.  
* tcg_exec_realizefn
    * Helper functions to check if ```cpu``` is correctly configured before starting tcg execution.
* tcg_exec_unrealizefn
    * Undo the ```cpu``` configuration.

# Coding Techniques I Learned
## ```inline``` Functions
There are 11 functions out of 31 in the ```cpu_exec.c``` file marked as ```inline```. Marking function as ```inline``` simply suggests compiler to replace the function call with the actual function code thus avoiding the overhead of function calls. 

## ```void *``` as Function Input
The ```tb_lookup_cmp``` function looks like:

    static bool tb_lookup_cmp(const void *p, const void *d)
    {
        const TranslationBlock *tb = p;
        const struct tb_desc *desc = d;

        /* ... */
    }

Reason for that is quite straight-forward, function ```tb_lookup_cmp``` is called as a function pointer in the following line,

    qht_lookup_custom(&tb_ctx.htable, &desc, h, tb_lookup_cmp);
The ```qht_lookup_custom``` function performs hash table lookups, and requires the subjects to be ```void``` typed for flexibility.

## Function Name Glueing
There' this function definition:

    const void *HELPER(lookup_tb_ptr)(CPUArchState *env)

The ```HELPER``` here is defined in a header file as 

    #define HELPER(name) glue(helper_, name)

The ```glue``` macro is defined in another header file as

    #define xglue(x, y) x ## y
    #define glue(x, y) xglue(x, y)
In this way, when compiled, the name of this function will be ```helper_lookup_tb_ptr```. Using macro makes it easier to change the prefix for a group of functions at once.

## Branch Prediction Optimization
A pair of macro is defined as a hint for compilers. Example usage:

    static inline TranslationBlock *tb_lookup(CPUState *cpu, vaddr pc, uint64_t cs_base, uint32_t flags, uint32_t cflags)
    {
        /* ... */
        if (likely(tb &&
                jc->array[hash].pc == pc &&
                tb->cs_base == cs_base &&
                tb->flags == flags &&
                tb_cflags(tb) == cflags)) {
            goto hit;
        }
        /* ... */
    hit:
        assert((tb_cflags(tb) & CF_PCREL) || tb->pc == pc);
        return tb;
    }

The macro definitions in a header file called ```compiler.h```:

    #ifndef likely
    #define likely(x)   __builtin_expect(!!(x), 1)
    #define unlikely(x)   __builtin_expect(!!(x), 0)
    #endif