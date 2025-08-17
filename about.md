---
layout: default
title: About Me
permalink: /about/
---

<style>
    .about-container {
        max-width: 1000px;
        margin: 0 auto;
        padding: 40px 20px;
        background-color: white;
        box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
        min-height: 100vh;
        font-family: 'Georgia', 'Times New Roman', serif;
        line-height: 1.7;
        color: #2c3e50;
        font-size: 16px;
    }

    .header {
        text-align: center;
        border-bottom: 3px solid #34495e;
        padding-bottom: 30px;
        margin-bottom: 40px;
    }

    .profile-image {
        width: 120px;
        height: 120px;
        border-radius: 50%;
        border: 4px solid #34495e;
        object-fit: cover;
        margin-bottom: 20px;
    }

    .profile-fallback {
        width: 120px;
        height: 120px;
        border-radius: 50%;
        background-color: #34495e;
        color: white;
        font-size: 36px;
        font-weight: bold;
        text-align: center;
        line-height: 120px;
        margin: 0 auto 20px auto;
        display: none;
    }

    .about-container h1 {
        font-size: 2.2em;
        color: #2c3e50;
        margin-bottom: 10px;
        font-weight: 700;
        letter-spacing: 0.5px;
    }

    .title {
        font-size: 1.3em;
        color: #7f8c8d;
        margin-bottom: 15px;
        font-style: italic;
    }

    .affiliation {
        font-size: 1.1em;
        color: #34495e;
        margin-bottom: 20px;
        font-weight: 600;
    }

    .bio {
        font-size: 1.05em;
        text-align: justify;
        margin-bottom: 40px;
        padding: 25px;
        background-color: #f8f9fa;
        border-left: 4px solid #34495e;
        border-radius: 0 8px 8px 0;
    }

    .section {
        margin-bottom: 40px;
    }

    .section-title {
        font-size: 1.5em;
        color: #2c3e50;
        margin-bottom: 20px;
        padding-bottom: 10px;
        border-bottom: 2px solid #ecf0f1;
        font-weight: 600;
        text-transform: uppercase;
        letter-spacing: 1px;
    }

    .experience-item {
        margin-bottom: 25px;
        padding: 20px;
        background-color: #f8f9fa;
        border-radius: 6px;
        border-left: 4px solid #34495e;
        transition: background-color 0.3s ease;
    }

    .experience-item:hover {
        background-color: #ecf0f1;
    }

    .job-title {
        font-size: 1.2em;
        font-weight: 600;
        color: #2c3e50;
        margin-bottom: 5px;
    }

    .institution {
        font-size: 1.05em;
        color: #34495e;
        font-weight: 500;
        margin-bottom: 5px;
    }

    .date-location {
        color: #7f8c8d;
        font-size: 0.95em;
        font-style: italic;
        margin-bottom: 8px;
    }

    .description {
        color: #555;
        font-size: 0.95em;
        line-height: 1.6;
    }

    .contact-section {
        background-color: #34495e;
        color: white;
        padding: 30px;
        border-radius: 8px;
        text-align: center;
        margin-top: 40px;
    }

    .contact-section h2 {
        margin-bottom: 15px;
        font-size: 1.4em;
    }

    .contact-email {
        display: inline-block;
        background-color: rgba(255, 255, 255, 0.1);
        padding: 12px 25px;
        border-radius: 4px;
        color: white;
        text-decoration: none;
        font-weight: 500;
        transition: background-color 0.3s ease;
    }

    .contact-email:hover {
        background-color: rgba(255, 255, 255, 0.2);
        text-decoration: none;
        color: white;
    }

    .research-interests {
        background-color: #f8f9fa;
        padding: 20px;
        border-radius: 6px;
        margin-bottom: 30px;
    }

    .research-interests h3 {
        color: #2c3e50;
        margin-bottom: 10px;
        font-size: 1.2em;
    }

    .research-interests ul {
        list-style-type: none;
        padding-left: 0;
    }

    .research-interests li {
        padding: 5px 0;
        color: #555;
        position: relative;
        padding-left: 20px;
    }

    .research-interests li:before {
        content: "•";
        color: #34495e;
        font-weight: bold;
        position: absolute;
        left: 0;
    }

    @media (max-width: 768px) {
        .about-container {
            padding: 20px 15px;
        }
        
        .about-container h1 {
            font-size: 1.8em;
        }
        
        .section-title {
            font-size: 1.3em;
        }
        
        .bio {
            padding: 20px;
        }
    }

    .publications-note {
        font-size: 0.9em;
        color: #7f8c8d;
        font-style: italic;
        margin-top: 10px;
    }
</style>

<div class="about-container">
    <div class="header">
        <img src="https://erkaiyublog.github.io/images/me.jpeg" alt="Erkai Yu" class="profile-image" onerror="this.style.display='none'; this.nextElementSibling.style.display='block';">
        <div class="profile-fallback">EY</div>
        <h1>Erkai Yu</h1>
        <div class="title">Computer Science Graduate Student</div>
        <div class="affiliation">University of Illinois Urbana-Champaign</div>
    </div>

    <div class="bio">
        <p>I am a second-year master's student in Computer Science at the University of Illinois Urbana-Champaign. My research focuses on software testing and operating system testing under the supervision of Professor <a href="https://mir.cs.illinois.edu/marinov/" target="_blank" style="color: #34495e; text-decoration: underline;">Darko Marinov</a>.</p>
    </div>

    <div class="section">
        <h2 class="section-title">Research Interests</h2>
        <div class="research-interests">
            <h3>Primary Research Areas:</h3>
            <ul>
                <li>Software Testing and Verification</li>
                <li>Operating System Testing</li>
                <li>Program Analysis</li>
                <li>System Security</li>
            </ul>
        </div>
    </div>

    <div class="section">
        <h2 class="section-title">Professional Experience</h2>
        
        <div class="experience-item">
            <div class="job-title">Research and Development Intern</div>
            <div class="institution">Momenta</div>
            <div class="date-location">February 2024 - June 2024 • Shanghai, China</div>
            <div class="description">
                Conducted research and development in autonomous driving technology, focusing on software testing and system validation.
            </div>
        </div>
    </div>

    <div class="section">
        <h2 class="section-title">Teaching Experience</h2>
        
        <div class="experience-item">
            <div class="job-title">Teaching Assistant</div>
            <div class="institution">CS 101: Introduction to Programming for Scientists and Engineers</div>
            <div class="date-location">University of Illinois Urbana-Champaign • Fall 2024, Spring 2025, Fall 2025</div>
            <div class="description">
                Assisted in course instruction, grading, and student support for introductory programming course.
            </div>
        </div>

        <div class="experience-item">
            <div class="job-title">Teaching Assistant</div>
            <div class="institution">ECE 220: Computer Systems and Programming</div>
            <div class="date-location">ZJUI • Spring 2024</div>
            <div class="description">
                Provided instructional support for computer systems and programming course.
            </div>
        </div>

        <div class="experience-item">
            <div class="job-title">Teaching Assistant</div>
            <div class="institution">CS 101: Introduction to Programming for Scientists and Engineers</div>
            <div class="date-location">ZJUI • Fall 2023</div>
            <div class="description">
                Assisted in teaching introductory programming concepts and provided student guidance.
            </div>
        </div>

        <div class="experience-item">
            <div class="job-title">Course Assistant</div>
            <div class="institution">ECE 391: Computer Systems Engineering</div>
            <div class="date-location">University of Illinois Urbana-Champaign • Spring 2023</div>
            <div class="description">
                Supported course operations and provided technical assistance for computer systems engineering course.
            </div>
        </div>
    </div>

    <div class="section">
        <h2 class="section-title">Education</h2>
        <div class="experience-item">
            <div class="job-title">Master of Science in Computer Science</div>
            <div class="institution">University of Illinois Urbana-Champaign</div>
            <div class="date-location">2023 - Present</div>
            <div class="description">
                Focus on software testing, operating systems, and program analysis.
            </div>
        </div>
    </div>

    <div class="contact-section">
        <h2>Contact Information</h2>
        <p style="margin-bottom: 20px; opacity: 0.9;">For research collaborations, academic discussions, or inquiries:</p>
        <a href="mailto:erkaiyu2@illinois.edu" class="contact-email">
            erkaiyu2@illinois.edu
        </a>
        <div class="publications-note">
            For a complete list of publications and research work, please visit my <a href="/" style="color: #bdc3c7; text-decoration: underline;">blog</a>.
        </div>
    </div>
</div>