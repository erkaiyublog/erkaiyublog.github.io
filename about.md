---
layout: default
title: About Me
permalink: /about/
---
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>About Me</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
            line-height: 1.6;
            color: #333;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
        }

        .container {
            max-width: 900px;
            margin: 0 auto;
            padding: 40px 20px;
        }

        .card {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            margin-bottom: 30px;
            transform: translateY(20px);
            opacity: 0;
            animation: slideUp 0.8s ease-out forwards;
        }

        @keyframes slideUp {
            to {
                transform: translateY(0);
                opacity: 1;
            }
        }

        .hero-section {
            text-align: center;
            margin-bottom: 40px;
        }

        .profile-container {
            position: relative;
            display: inline-block;
            margin-bottom: 30px;
        }

        .profile-image {
            width: 150px;
            height: 150px;
            border-radius: 50%;
            border: 5px solid #667eea;
            object-fit: cover;
            transition: transform 0.3s ease;
        }

        .profile-image:hover {
            transform: scale(1.1) rotate(5deg);
        }

        .profile-placeholder {
            width: 150px;
            height: 150px;
            border-radius: 50%;
            background: linear-gradient(135deg, #667eea, #764ba2);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 48px;
            color: white;
            font-weight: bold;
            margin: 0 auto;
        }

        h1 {
            font-size: 2.5em;
            color: #2c3e50;
            margin-bottom: 10px;
            background: linear-gradient(135deg, #667eea, #764ba2);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .subtitle {
            font-size: 1.3em;
            color: #7f8c8d;
            margin-bottom: 30px;
        }

        .bio {
            font-size: 1.1em;
            text-align: left;
            margin-bottom: 30px;
            padding: 20px;
            background: linear-gradient(135deg, rgba(102, 126, 234, 0.1), rgba(118, 75, 162, 0.1));
            border-radius: 15px;
            border-left: 4px solid #667eea;
        }

        .section-title {
            font-size: 1.8em;
            color: #2c3e50;
            margin-bottom: 20px;
            display: flex;
            align-items: center;
            gap: 15px;
        }

        .section-icon {
            width: 30px;
            height: 30px;
            background: linear-gradient(135deg, #667eea, #764ba2);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: bold;
        }

        .experience-item {
            background: #f8f9fa;
            border-radius: 15px;
            padding: 25px;
            margin-bottom: 20px;
            border-left: 4px solid #667eea;
            transition: transform 0.3s ease, box-shadow 0.3s ease;
            cursor: pointer;
        }

        .experience-item:hover {
            transform: translateX(10px);
            box-shadow: 0 10px 25px rgba(102, 126, 234, 0.2);
        }

        .job-title {
            font-size: 1.3em;
            font-weight: bold;
            color: #2c3e50;
            margin-bottom: 5px;
        }

        .company {
            font-size: 1.1em;
            color: #667eea;
            font-weight: 600;
            margin-bottom: 5px;
        }

        .date-location {
            color: #7f8c8d;
            font-size: 0.95em;
            margin-bottom: 10px;
        }

        .contact-section {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border-radius: 20px;
            padding: 30px;
            text-align: center;
        }

        .contact-link {
            display: inline-block;
            background: rgba(255, 255, 255, 0.2);
            padding: 15px 30px;
            border-radius: 50px;
            color: white;
            text-decoration: none;
            font-weight: 600;
            transition: all 0.3s ease;
            backdrop-filter: blur(10px);
        }

        .contact-link:hover {
            background: rgba(255, 255, 255, 0.3);
            transform: translateY(-3px);
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
        }

        .floating-elements {
            position: fixed;
            width: 100%;
            height: 100%;
            top: 0;
            left: 0;
            pointer-events: none;
            z-index: -1;
        }

        .floating-circle {
            position: absolute;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.1);
            animation: float 6s ease-in-out infinite;
        }

        .floating-circle:nth-child(1) {
            width: 80px;
            height: 80px;
            top: 20%;
            left: 10%;
            animation-delay: 0s;
        }

        .floating-circle:nth-child(2) {
            width: 60px;
            height: 60px;
            top: 60%;
            right: 10%;
            animation-delay: 2s;
        }

        .floating-circle:nth-child(3) {
            width: 40px;
            height: 40px;
            bottom: 20%;
            left: 20%;
            animation-delay: 4s;
        }

        @keyframes float {
            0%, 100% { transform: translateY(0px) rotate(0deg); }
            50% { transform: translateY(-20px) rotate(180deg); }
        }

        @media (max-width: 768px) {
            .container {
                padding: 20px 15px;
            }
            
            .card {
                padding: 25px;
            }
            
            h1 {
                font-size: 2em;
            }
            
            .section-title {
                font-size: 1.5em;
            }
        }

        .typing-animation {
            overflow: hidden;
            border-right: 3px solid #667eea;
            white-space: nowrap;
            animation: typing 3s steps(40) 1s forwards, blink 1s infinite;
            width: 0;
        }

        @keyframes typing {
            to { width: 100%; }
        }

        @keyframes blink {
            50% { border-color: transparent; }
        }
    </style>
</head>
<body>
    <div class="floating-elements">
        <div class="floating-circle"></div>
        <div class="floating-circle"></div>
        <div class="floating-circle"></div>
    </div>

    <div class="container">
        <div class="card">
            <div class="hero-section">
                <div class="profile-container">
                    <img src="https://github.com/erkaiyublog/erkaiyublog.github.io/blob/master/images/me.jpeg" alt="Erkai Yu" class="profile-image">
                </div>
                <h1>Erkai Yu</h1>
                <div class="subtitle typing-animation">Computer Science Graduate Student</div>
            </div>

            <div class="bio">
                <p>I'm a second-year master's student majoring in computer science at UIUC. I'm currently conducting research on software testing and operating system testing under the supervision of my advisor, Professor <a href="https://mir.cs.illinois.edu/marinov/" target="_blank" style="color: #667eea; text-decoration: none; font-weight: 600;">Darko Marinov</a>.</p>
            </div>
        </div>

        <div class="card">
            <h2 class="section-title">
                <div class="section-icon">💼</div>
                Working Experience
            </h2>
            
            <div class="experience-item" onclick="toggleDetails(this)">
                <div class="job-title">Research and Development Intern</div>
                <div class="company">Momenta</div>
                <div class="date-location">Feb 2024 - Jun 2024 • Shanghai, China</div>
            </div>
        </div>

        <div class="card">
            <h2 class="section-title">
                <div class="section-icon">🎓</div>
                Teaching Experience
            </h2>
            
            <div class="experience-item" onclick="toggleDetails(this)">
                <div class="job-title">Teaching Assistant</div>
                <div class="company">CS 101: Introduction to Programming for Scientists and Engineers, UIUC</div>
                <div class="date-location">FA24, SP25, FA25</div>
            </div>

            <div class="experience-item" onclick="toggleDetails(this)">
                <div class="job-title">Teaching Assistant</div>
                <div class="company">ECE 220: Computer Systems and Programming, ZJUI</div>
                <div class="date-location">SP24</div>
            </div>

            <div class="experience-item" onclick="toggleDetails(this)">
                <div class="job-title">Teaching Assistant</div>
                <div class="company">CS 101: Introduction to Programming for Scientists and Engineers, ZJUI</div>
                <div class="date-location">FA23</div>
            </div>

            <div class="experience-item" onclick="toggleDetails(this)">
                <div class="job-title">Course Assistant</div>
                <div class="company">ECE 391: Computer Systems Engineering, UIUC</div>
                <div class="date-location">SP23</div>
            </div>
        </div>

        <div class="card contact-section">
            <h2 style="margin-bottom: 20px; font-size: 1.8em;">Let's Connect!</h2>
            <p style="margin-bottom: 25px; opacity: 0.9;">Feel free to reach out for research collaborations, discussions, or just to say hello!</p>
            <a href="mailto:erkaiyu2@illinois.edu" class="contact-link">
                📧 erkaiyu2@illinois.edu
            </a>
        </div>
    </div>

    <script>
        // Add staggered animation delays
        document.addEventListener('DOMContentLoaded', function() {
            const cards = document.querySelectorAll('.card');
            cards.forEach((card, index) => {
                card.style.animationDelay = `${index * 0.2}s`;
            });

            // Restart typing animation periodically
            setTimeout(function() {
                const typingElement = document.querySelector('.typing-animation');
                setInterval(function() {
                    typingElement.style.animation = 'none';
                    setTimeout(function() {
                        typingElement.style.animation = 'typing 3s steps(40) forwards, blink 1s infinite';
                    }, 100);
                }, 8000);
            }, 4000);
        });

        function toggleDetails(element) {
            element.style.transform = element.style.transform === 'scale(1.02)' ? '' : 'scale(1.02)';
        }

        // Add subtle parallax effect
        window.addEventListener('scroll', function() {
            const scrolled = window.pageYOffset;
            const parallax = document.querySelector('.floating-elements');
            const speed = scrolled * 0.5;
            parallax.style.transform = `translateY(${speed}px)`;
        });
    </script>
</body>
</html>