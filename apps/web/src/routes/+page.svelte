<script>
    import { onMount } from 'svelte';

    let navbar;

    onMount(() => {
        // Top Navigation Blur effect on scroll
        const handleScroll = () => {
            if (window.scrollY > 20) {
                navbar.classList.add('scrolled');
            } else {
                navbar.classList.remove('scrolled');
            }
        };

        window.addEventListener('scroll', handleScroll);

        // Intersection Observer for scroll animations
        const observerOptions = {
            root: null,
            rootMargin: '0px',
            threshold: 0.15
        };

        const observer = new IntersectionObserver((entries, observer) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    entry.target.classList.add('visible');
                    // Optional: Stop observing once the animation has played
                    // observer.unobserve(entry.target);
                }
            });
        }, observerOptions);

        // Select all elements with animation classes
        const fadeElements = document.querySelectorAll('.fade-in-up, .fade-in-left, .fade-in-right');
        
        fadeElements.forEach(el => {
            observer.observe(el);
        });

        // Simple smooth scrolling for anchor links 
        document.querySelectorAll('.nav-links a[href^="#"]').forEach(anchor => {
            anchor.addEventListener('click', function (e) {
                e.preventDefault();
                const targetId = this.getAttribute('href');
                if(targetId === '#') return;
                
                const targetElement = document.querySelector(targetId);
                if (targetElement) {
                    // Adjust scroll position accounting for fixed navbar
                    const navbarHeight = navbar.offsetHeight;
                    const elementPosition = targetElement.getBoundingClientRect().top;
                    const offsetPosition = elementPosition + window.pageYOffset - navbarHeight - 40; // 40px extra buffer
                    
                    window.scrollTo({
                        top: offsetPosition,
                        behavior: 'smooth'
                    });
                }
            });
        });

        // Micro-interaction for feature cards - slight 3D tilt effect on mouse move
        const featureCards = document.querySelectorAll('.feature-card');
        
        featureCards.forEach(card => {
            card.addEventListener('mousemove', (e) => {
                const rect = card.getBoundingClientRect();
                // Calculate mouse position relative to the card center
                const x = e.clientX - rect.left; // x position within the element.
                const y = e.clientY - rect.top;  // y position within the element.
                
                const centerX = rect.width / 2;
                const centerY = rect.height / 2;
                
                const rotateX = ((y - centerY) / centerY) * -5; // Max 5 degrees tilt
                const rotateY = ((x - centerX) / centerX) * 5;
                
                card.style.transform = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) translateY(-5px)`;
            });
            
            card.addEventListener('mouseleave', () => {
                // Reset transform when mouse leaves
                card.style.transform = 'perspective(1000px) rotateX(0) rotateY(0) translateY(0)';
                // Give it a smooth transition back to initial state
                card.style.transition = 'transform 0.5s ease-out';
                
                setTimeout(() => {
                    // Remove transition to not interfere with hover states
                    card.style.transition = '';
                }, 500);
            });
        });

        return () => {
            window.removeEventListener('scroll', handleScroll);
        };
    });
</script>

<svelte:head>
    <title>Nullspace | AI-Native Research Workspace</title>
    <meta name="description" content="The radically fast, AI-native research workspace. Compile LaTeX in <3s and write papers flawlessly.">
</svelte:head>

<!-- Background Glow Effects -->
<div class="ambient-glow glow-top"></div>
<div class="ambient-glow glow-right"></div>

<nav bind:this={navbar} class="navbar" id="navbar">
    <div class="nav-container">
        <a href="#" class="brand">
            <span class="text-primary">null</span>space
        </a>
        <div class="nav-links">
            <a href="#features">Features</a>
            <a href="#compiler">Speed</a>
            <a href="#agents">AI Agents</a>
            <a href="#manifesto" class="text-muted">Manifesto</a>
        </div>
        <div class="nav-actions">
            <a href="#" class="btn btn-ghost">Sign In</a>
            <a href="#" class="btn btn-primary">Get Early Access</a>
        </div>
    </div>
</nav>

<main>
    <!-- Hero Section -->
    <section class="hero fade-in-up">
        <div class="hero-content">
            <div class="badge">
                <span class="pulse-dot"></span>
                Now in Private Beta
            </div>
            <h1 class="hero-title">
                The research workspace <br> built for <span class="text-highlight">speed</span>.
            </h1>
            <p class="hero-subtitle">
                Compile LaTeX in &lt;3 seconds. Organize chaotic PDFs instantly. Let context-aware AI agents draft your next big breakthrough. We eliminated the bloat, so you can focus on the science.
            </p>
            <div class="hero-cta">
                <a href="#" class="btn btn-primary btn-lg">Start Writing for Free</a>
                <a href="#" class="btn btn-secondary btn-lg">
                    <i class="ph ph-play-circle"></i> Watch Demo
                </a>
            </div>
            <!-- Feature Highlight Badges -->
            <div class="tech-stack">
                <span class="tech-badge"><i class="ph ph-lightning text-primary"></i> <span class="text-muted">&lt;3s Compilations</span></span>
                <span class="tech-badge"><i class="ph ph-users-three text-primary"></i> <span class="text-muted">Realtime Collaboration</span></span>
                <span class="tech-badge"><i class="ph ph-magic-wand text-primary"></i> <span class="text-muted">Agentic Drafting</span></span>
            </div>
        </div>

        <!-- Dashboard Mockup Graphic -->
        <div class="dashboard-mockup glass-panel fade-in-up delay-1">
            <div class="mockup-header">
                <div class="window-controls">
                    <span></span><span></span><span></span>
                </div>
                <div class="mockup-title">master_plan.tex &mdash; Nullspace Editor</div>
            </div>
            <div class="mockup-body">
                <div class="mockup-sidebar">
                    <div class="mockup-line skeleton w-100"></div>
                    <div class="mockup-line skeleton w-80"></div>
                    <div class="mockup-line skeleton w-90"></div>
                    <div class="mockup-line skeleton w-60"></div>
                    <div class="mockup-agent-box">
                        <i class="ph ph-sparkle text-primary"></i>
                        <div class="mockup-line skeleton w-80 m-0"></div>
                    </div>
                </div>
                <div class="mockup-editor">
                    <span class="code-keyword">\documentclass</span>&lbrace;<span class="code-string">article</span>&rbrace;<br><br>
                    <span class="code-keyword">\title</span>&lbrace;The Future of AI-Native Workspaces&rbrace;<br>
                    <span class="code-keyword">\author</span>&lbrace;Nullspace Team&rbrace;<br><br>
                    <span class="code-keyword">\begin</span>&lbrace;<span class="code-string">document</span>&rbrace;<br>
                    <span class="code-keyword">\maketitle</span><br><br>
                    <span class="code-comment">% The AI agent automatically parsed 5 PDFs to draft this introduction</span><br>
                    Traditional LaTeX editors introduce severe cognitive friction through slow compilation cycles and disconnected reference management...
                    <div class="cursor-blink"></div>
                </div>
                <div class="mockup-pdf">
                    <div class="pdf-page">
                        <h3 class="pdf-title">The Future of AI-Native Workspaces</h3>
                        <p class="pdf-author">Nullspace Team</p>
                        <div class="mockup-line skeleton w-100 mt-4"></div>
                        <div class="mockup-line skeleton w-100"></div>
                        <div class="mockup-line skeleton w-80"></div>
                    </div>
                </div>
            </div>
        </div>
    </section>

    <!-- Features Grid Selection -->
    <section id="features" class="features-section">
        <div class="section-header text-center fade-in-up">
            <h2 class="section-title">A true technical differentiator.</h2>
            <p class="section-subtitle">We didn't just build another editor. We re-engineered the entire LaTeX pipeline from the ground up for zero-latency collaboration.</p>
        </div>

        <div class="features-grid">
            <!-- Feature 1 -->
            <div class="feature-card glass-panel fade-in-up">
                <div class="feature-icon bg-primary-light">
                    <i class="ph ph-lightning text-primary"></i>
                </div>
                <h3 class="feature-title">&lt;3s Compilations</h3>
                <p class="feature-desc">Instant feedback on every keystroke. Say goodbye to bottlenecks and never watch a loading spinner again.</p>
            </div>

            <!-- Feature 2 -->
            <div class="feature-card glass-panel fade-in-up delay-1">
                <div class="feature-icon bg-primary-light">
                    <i class="ph ph-users-three text-primary"></i>
                </div>
                <h3 class="feature-title">Realtime Collaboration</h3>
                <p class="feature-desc">Write together seamlessly. See your co-authors' cursors fly across the document with absolutely zero lag.</p>
            </div>

            <!-- Feature 3 -->
            <div class="feature-card glass-panel fade-in-up delay-2">
                <div class="feature-icon bg-primary-light">
                    <i class="ph ph-brain text-primary"></i>
                </div>
                <h3 class="feature-title">Intelligent Assistance</h3>
                <p class="feature-desc">Drag and drop PDFs, voice memos, and images. The built-in assistant automatically extracts key insights and drafts your sections.</p>
            </div>
        </div>
    </section>

    <!-- Deep Dive AI Agents -->
    <section id="agents" class="agents-section">
        <div class="agents-container">
            <div class="agents-content fade-in-left">
                <div class="badge badge-secondary mb-4">
                    <i class="ph ph-magic-wand"></i> Agentic Workflow
                </div>
                <h2 class="section-title">Your superhuman research assistant.</h2>
                <p class="section-subtitle">Nullspace doesn't just autocomplete text. It understands your entire project context via semantic analysis.</p>
                
                <ul class="agent-list">
                    <li>
                        <i class="ph-fill ph-check-circle text-primary"></i>
                        <div>
                            <strong>Structure Proposal Agent</strong>
                            <span>Analyzes your messy notes and proposes NeurIPS/IEEE paper structures.</span>
                        </div>
                    </li>
                    <li>
                        <i class="ph-fill ph-check-circle text-primary"></i>
                        <div>
                            <strong>Citation Agent</strong>
                            <span>Paste a DOI or abstract; it instantly generates valid BibTeX and interpolates citations.</span>
                        </div>
                    </li>
                    <li>
                        <i class="ph-fill ph-check-circle text-primary"></i>
                        <div>
                            <strong>Consistency Guardian</strong>
                            <span>A background loop that catches discrepancies between your claims and your data figures.</span>
                        </div>
                    </li>
                </ul>
            </div>
            <div class="agents-visual fade-in-right">
                <div class="glass-panel agent-chat">
                    <div class="chat-message agent">
                        <div class="avatar"><i class="ph-fill ph-sparkle"></i></div>
                        <div class="msg-bubble">I've analyzed the 3 new dataset PDFs you dropped in. Would you like me to draft the "Related Work" section comparing them to our baseline?</div>
                    </div>
                    <div class="chat-message user">
                        <div class="avatar bg-dark"><i class="ph-fill ph-user"></i></div>
                        <div class="msg-bubble">Yes, make sure to emphasize our novel latency improvements.</div>
                    </div>
                    <div class="chat-message agent">
                        <div class="avatar"><i class="ph-fill ph-sparkle"></i></div>
                        <div class="msg-bubble system-action">
                            <i class="ph-fill ph-file-code"></i> Generating `related_work.tex`...
                            <div class="progress-bar mt-2"><div class="progress-fill"></div></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
    
    <!-- CTA Section -->
    <section class="cta-section text-center fade-in-up">
        <h2 class="section-title">Ready to write at the speed of thought?</h2>
        <p class="section-subtitle mb-8">Join the private beta. Experience the fastest, smartest LaTeX editor ever built.</p>
        <a href="#" class="btn btn-primary btn-lg pulse-shadow">Claim Your Invite</a>
    </section>
</main>

<footer>
    <div class="footer-content">
        <div class="footer-brand">
            <a href="#" class="brand">
                <span class="text-primary">null</span>space
            </a>
            <p class="text-sm text-muted mt-2">The AI-native research workspace built for speed.</p>
        </div>
        <div class="footer-links">
            <div class="link-group">
                <h4>Product</h4>
                <a href="#">Features</a>
                <a href="#">Pricing</a>
                <a href="#">Changelog</a>
            </div>
            <div class="link-group">
                <h4>Resources</h4>
                <a href="#">Documentation</a>
                <a href="#">Manifesto</a>
                <a href="#">LaTeX Guide</a>
            </div>
            <div class="link-group">
                <h4>Company</h4>
                <a href="#">About</a>
                <a href="#">Twitter</a>
                <a href="#">Contact</a>
            </div>
        </div>
    </div>
    <div class="footer-bottom">
        <p class="text-sm text-muted">&copy; 2026 Nullspace Inc. All rights reserved.</p>
    </div>
</footer>
