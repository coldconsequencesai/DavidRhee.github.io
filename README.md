// AIAA — David's AI Automation Agency
// Complete Framer Website — FULLY CUSTOMIZABLE
// Every single element editable from Framer property controls

import { useState, useEffect, useRef, useCallback } from "react"
import { motion, AnimatePresence, useInView } from "framer-motion"
import { addPropertyControls, ControlType } from "framer"

// ============================================
// TYPES
// ============================================
interface CaseStudy {
    tag: string
    title: string
    industry: string
    scope: string
    challenge: string
    solution: string
    outcome: string
}

interface BlogArticle {
    title: string
    date: string
    body: string
}

// ============================================
// FADE-IN WRAPPER
// ============================================
function FadeIn({
    children,
    delay = 0,
    direction = "up",
}: {
    children: React.ReactNode
    delay?: number
    direction?: "up" | "left" | "right" | "none"
}) {
    const ref = useRef(null)
    const isInView = useInView(ref, { once: true, margin: "-50px" })
    const variants = {
        hidden: {
            opacity: 0,
            x: direction === "left" ? -40 : direction === "right" ? 40 : 0,
            y: direction === "up" ? 30 : 0,
        },
        visible: {
            opacity: 1,
            x: 0,
            y: 0,
            transition: {
                duration: 0.7,
                delay,
                ease: [0.25, 0.46, 0.45, 0.94],
            },
        },
    }
    return (
        <motion.div
            ref={ref}
            variants={variants}
            initial="hidden"
            animate={isInView ? "visible" : "hidden"}
            style={{ width: "100%" }}
        >
            {children}
        </motion.div>
    )
}

// ============================================
// ANIMATED COUNTER
// ============================================
function AnimatedCounter({
    target,
    suffix = "",
}: {
    target: number
    suffix?: string
}) {
    const [count, setCount] = useState(0)
    const ref = useRef(null)
    const isInView = useInView(ref, { once: true })
    useEffect(() => {
        if (!isInView) return
        let current = 0
        const increment = Math.ceil(target / 40)
        const timer = setInterval(() => {
            current += increment
            if (current >= target) {
                setCount(target)
                clearInterval(timer)
            } else {
                setCount(current)
            }
        }, 40)
        return () => clearInterval(timer)
    }, [isInView, target])
    return (
        <span ref={ref}>
            {count}
            {suffix}
        </span>
    )
}

// ============================================
// PARTICLE CANVAS
// ============================================
function ParticleCanvas({ color }: { color: string }) {
    const canvasRef = useRef<HTMLCanvasElement>(null)
    useEffect(() => {
        const canvas = canvasRef.current
        if (!canvas) return
        const ctx = canvas.getContext("2d")
        if (!ctx) return
        let animationId: number
        let particles: any[] = []
        const hexToRgb = (hex: string) => {
            const r = parseInt(hex.slice(1, 3), 16)
            const g = parseInt(hex.slice(3, 5), 16)
            const b = parseInt(hex.slice(5, 7), 16)
            return `${r}, ${g}, ${b}`
        }
        const rgb = hexToRgb(color)
        function resize() {
            const parent = canvas!.parentElement
            if (!parent) return
            const rect = parent.getBoundingClientRect()
            canvas!.width = rect.width * window.devicePixelRatio
            canvas!.height = rect.height * window.devicePixelRatio
            canvas!.style.width = rect.width + "px"
            canvas!.style.height = rect.height + "px"
            ctx!.scale(window.devicePixelRatio, window.devicePixelRatio)
            createParticles(rect.width, rect.height)
        }
        function createParticles(w: number, h: number) {
            particles = []
            const count = Math.min(Math.floor((w * h) / 15000), 60)
            for (let i = 0; i < count; i++) {
                particles.push({
                    x: Math.random() * w,
                    y: Math.random() * h,
                    vx: (Math.random() - 0.5) * 0.3,
                    vy: (Math.random() - 0.5) * 0.3,
                    radius: Math.random() * 1.5 + 0.5,
                    opacity: Math.random() * 0.4 + 0.1,
                })
            }
        }
        function draw() {
            const parent = canvas!.parentElement
            if (!parent) return
            const rect = parent.getBoundingClientRect()
            const w = rect.width
            const h = rect.height
            ctx!.clearRect(0, 0, w, h)
            for (let i = 0; i < particles.length; i++) {
                for (let j = i + 1; j < particles.length; j++) {
                    const dx = particles[i].x - particles[j].x
                    const dy = particles[i].y - particles[j].y
                    const dist = Math.sqrt(dx * dx + dy * dy)
                    if (dist < 150) {
                        const opacity = (1 - dist / 150) * 0.08
                        ctx!.beginPath()
                        ctx!.strokeStyle = `rgba(${rgb}, ${opacity})`
                        ctx!.lineWidth = 0.5
                        ctx!.moveTo(particles[i].x, particles[i].y)
                        ctx!.lineTo(particles[j].x, particles[j].y)
                        ctx!.stroke()
                    }
                }
            }
            particles.forEach((p) => {
                ctx!.beginPath()
                ctx!.arc(p.x, p.y, p.radius, 0, Math.PI * 2)
                ctx!.fillStyle = `rgba(${rgb}, ${p.opacity})`
                ctx!.fill()
                p.x += p.vx
                p.y += p.vy
                if (p.x < 0 || p.x > w) p.vx *= -1
                if (p.y < 0 || p.y > h) p.vy *= -1
            })
            animationId = requestAnimationFrame(draw)
        }
        resize()
        draw()
        window.addEventListener("resize", resize)
        return () => {
            cancelAnimationFrame(animationId)
            window.removeEventListener("resize", resize)
        }
    }, [color])
    return (
        <canvas
            ref={canvasRef}
            style={{
                position: "absolute",
                top: 0,
                left: 0,
                width: "100%",
                height: "100%",
            }}
        />
    )
}

// ============================================
// MOCK UI FOR PORTFOLIO
// ============================================
function MockUI({
    variant = 0,
    accentColor,
    bgColor,
    borderColor,
}: {
    variant?: number
    accentColor: string
    bgColor: string
    borderColor: string
}) {
    const patterns = [
        ["long", "medium", "short", "accent"],
        ["medium", "long", "accent", "short"],
        ["short", "long", "medium", "accent"],
        ["accent", "long", "short", "medium"],
        ["long", "accent", "medium", "short"],
        ["medium", "accent", "short", "long"],
        ["short", "accent", "long", "medium"],
        ["accent", "medium", "long", "accent"],
    ]
    const lines = patterns[variant % patterns.length]
    const widths: Record<string, string> = {
        short: "40%",
        medium: "65%",
        long: "85%",
        accent: "30%",
    }
    return (
        <div
            style={{
                width: "100%",
                height: "100%",
                display: "flex",
                alignItems: "center",
                justifyContent: "center",
                background: `linear-gradient(135deg, #1A1A1A, ${bgColor})`,
                position: "relative",
            }}
        >
            <div
                style={{
                    position: "absolute",
                    inset: 0,
                    backgroundImage: `linear-gradient(${accentColor}0D 1px, transparent 1px), linear-gradient(90deg, ${accentColor}0D 1px, transparent 1px)`,
                    backgroundSize: "30px 30px",
                }}
            />
            <div
                style={{
                    width: "70%",
                    height: "60%",
                    background: bgColor,
                    borderRadius: 8,
                    border: `1px solid ${borderColor}`,
                    position: "relative",
                    zIndex: 1,
                    padding: 12,
                }}
            >
                <div style={{ display: "flex", gap: 4, marginBottom: 10 }}>
                    {[0, 1, 2].map((i) => (
                        <div
                            key={i}
                            style={{
                                width: 6,
                                height: 6,
                                borderRadius: "50%",
                                background: borderColor,
                            }}
                        />
                    ))}
                </div>
                {lines.map((type, i) => (
                    <div
                        key={i}
                        style={{
                            height: 4,
                            width: widths[type],
                            background:
                                type === "accent"
                                    ? accentColor + "4D"
                                    : borderColor,
                            borderRadius: 2,
                            marginBottom: 6,
                        }}
                    />
                ))}
            </div>
        </div>
    )
}

// ============================================
// IMAGE OR PLACEHOLDER COMPONENT
// ============================================
function ImageOrPlaceholder({
    src,
    alt,
    placeholderIcon,
    placeholderText,
    style,
    accentColor,
    bgColor,
}: {
    src?: string
    alt?: string
    placeholderIcon?: string
    placeholderText?: string
    style?: React.CSSProperties
    accentColor: string
    bgColor: string
}) {
    if (src && src.trim() !== "") {
        return (
            <img
                src={src}
                alt={alt || ""}
                style={{
                    width: "100%",
                    height: "100%",
                    objectFit: "cover",
                    ...style,
                }}
            />
        )
    }
    return (
        <div
            style={{
                width: "100%",
                height: "100%",
                display: "flex",
                flexDirection: "column",
                alignItems: "center",
                justifyContent: "center",
                background: bgColor,
                ...style,
            }}
        >
            {placeholderIcon && (
                <div
                    style={{ fontSize: "3rem", marginBottom: 12, opacity: 0.3 }}
                >
                    {placeholderIcon}
                </div>
            )}
            {placeholderText && (
                <div
                    style={{
                        fontSize: "0.8rem",
                        color: "#666",
                        textAlign: "center",
                        padding: "0 16px",
                    }}
                >
                    {placeholderText}
                </div>
            )}
        </div>
    )
}

// ============================================
// MAIN COMPONENT
// ============================================
export default function AIAAWebsite(props: Record<string, any>) {
    const {
        // === GLOBAL COLORS ===
        accentColor = "#0066FF",
        accentHoverColor = "#0052CC",
        bgPrimary = "#0D0D0D",
        bgSecondary = "#111111",
        bgTertiary = "#1A1A1A",
        bgCard = "#161616",
        bgCardHover = "#1E1E1E",
        textPrimary = "#FFFFFF",
        textSecondary = "#B0B0B0",
        textMuted = "#666666",
        borderColor = "#222222",
        borderLightColor = "#333333",
        starColor = "#FFB800",
        gradientSecondary = "#00AAFF",

        // === GLOBAL TYPOGRAPHY ===
        headingFont = "'Space Grotesk', 'Inter', -apple-system, sans-serif",
        bodyFont = "'Inter', 'SF Pro Display', -apple-system, sans-serif",
        headingWeight = 700,
        bodyWeight = 400,
        baseFontSize = 16,

        // === GLOBAL SPACING ===
        sectionPadding = 120,
        containerMaxWidth = 1200,
        cardBorderRadius = 12,
        buttonBorderRadius = 50,
        navHeight = 72,

        // === GLOBAL IMAGES ===
        logoImage = "",
        faviconUrl = "",
        ogImage = "",

        // === NAV ===
        navBg = "rgba(13,13,13,0.9)",
        navBorderColor = "#222222",
        navLogoText = "AIAA",
        navLogoDot = ".",
        navLogoColor = "#FFFFFF",
        navLogoDotColor = "",
        navLogoFontSize = 1.5,
        navLogoFontWeight = 700,
        navLinkHome = "Home",
        navLinkServices = "Services",
        navLinkPortfolio = "Portfolio",
        navLinkAbout = "About",
        navLinkBlog = "Insights",
        navLinkContact = "Contact",
        navCtaText = "Book a Call",
        navCtaFontSize = 0.85,
        navLinkFontSize = 0.875,
        navLinkColor = "",
        navLinkActiveColor = "",
        navCtaBg = "",
        navCtaColor = "#FFFFFF",
        showNavCta = true,

        // === HERO ===
        heroTag = "AI Automation Agency — Los Angeles",
        heroTagIcon = "●",
        heroTagBg = "",
        heroTagBorderColor = "",
        heroTagColor = "",
        heroTagFontSize = 0.8,
        heroHeadline1 = "I Build the AI Systems That ",
        heroHighlight = "Run Your Business",
        heroHeadline2 = " While You Sleep.",
        heroHeadlineFontSize = "clamp(2.5rem, 5.5vw, 4.5rem)",
        heroHeadlineLetterSpacing = -2,
        heroHeadlineLineHeight = 1.08,
        heroHighlightGradient1 = "",
        heroHighlightGradient2 = "",
        heroSubheadline = "AI-powered automation, high-converting web platforms, and scalable digital infrastructure — engineered specifically for healthcare practices and elite real estate professionals.",
        heroSubFontSize = 1.2,
        heroSubColor = "",
        heroSubMaxWidth = 600,
        heroCta1Text = "See My Work",
        heroCta1Bg = "",
        heroCta1Color = "#FFFFFF",
        heroCta1FontSize = 0.95,
        heroCta1Icon = "→",
        showHeroCta1Icon = true,
        heroCta2Text = "Book a Strategy Call",
        heroCta2Bg = "transparent",
        heroCta2Color = "",
        heroCta2BorderColor = "",
        heroCta2FontSize = 0.95,
        heroGradientColor = "",
        heroGradientOpacity = 0.15,
        showParticles = true,
        heroBgImage = "",
        heroMinHeight = "calc(100vh - 72px)",
        heroContentAlign = "left",

        // === TRUST STRIP ===
        showTrustStrip = true,
        trustBg = "",
        trustLabel = "Systems Built For",
        trustLabelColor = "",
        trustLabelFontSize = 0.75,
        trust1 = "Classic Clinic",
        trust2 = "Dental Clinic of Compton",
        trust3 = "Dream Realty LA",
        trust4 = "Toma Barseghian",
        trust5 = "Frank Lopez",
        trust6 = "Alina Muradyan",
        trustNameColor = "",
        trustNameFontSize = 0.95,
        trustLogo1 = "",
        trustLogo2 = "",
        trustLogo3 = "",
        trustLogo4 = "",
        trustLogo5 = "",
        trustLogo6 = "",
        trustLogoHeight = 24,

        // === PROBLEMS SECTION ===
        showProblems = true,
        problemsBg = "",
        problemsLabel = "The Problem",
        problemsLabelColor = "",
        problemsTitle = "Your Business Is Bleeding Revenue. Here's Why.",
        problemsTitleColor = "",
        problemsTitleFontSize = "clamp(2rem, 4vw, 3rem)",
        problemsSubtitle = "Most high-value service businesses are losing thousands monthly to operational inefficiency. I fix that.",
        problemsSubColor = "",
        problem1Icon = "⏱️",
        problem1Title = "You're Losing Hours to Manual Tasks",
        problem1Body = "Every minute spent on manual follow-ups, missed calls, and scheduling chaos is revenue walking out the door. I replace that chaos with AI systems that execute flawlessly, 24/7.",
        problem1TitleColor = "",
        problem1BodyColor = "",
        problem2Icon = "📉",
        problem2Title = "You're Dropping Leads and Losing Patients",
        problem2Body = "Unanswered inquiries and slow response times are silently killing your growth. I build inbound workflow automation that guarantees zero dropped leads — every single time.",
        problem3Icon = "⭐",
        problem3Title = "Your Online Presence Isn't Working for You",
        problem3Body = "An outdated website and nonexistent review strategy cost you trust before a prospect ever picks up the phone. I engineer high-converting storefronts and automated reputation systems.",
        problemCardBg = "",
        problemCardBorder = "",
        problemIconBg = "",
        problemIconBorder = "",
        problemIconSize = 48,
        problemTitleFontSize = 1.25,
        problemBodyFontSize = 0.95,

        // === SERVICES OVERVIEW ===
        showServicesOverview = true,
        servicesOverviewBg = "",
        servicesLabel = "What I Build",
        servicesTitle = "End-to-End Systems That Scale",
        servicesSubtitle = "Automation, web platforms, content engines, and strategic consulting — all engineered for measurable revenue impact.",
        serviceCard1Icon = "🤖",
        serviceCard1Title = "AI Automation & Systems Engineering",
        serviceCard1Body = "Intelligent bots, automated scheduling, no-show prevention, inbound triage, and review generation — all built on enterprise-grade platforms.",
        serviceCard1Image = "",
        serviceCard2Icon = "🌐",
        serviceCard2Title = "Custom Web Design & Digital Infrastructure",
        serviceCard2Body = "High-converting websites purpose-built for healthcare practices and luxury real estate professionals.",
        serviceCard2Image = "",
        serviceCard3Icon = "🎬",
        serviceCard3Title = "High-Volume Content Strategy & Digital Media",
        serviceCard3Body = "Scalable content pipelines powered by AI — from video production to synthetic audio and music generation.",
        serviceCard3Image = "",
        serviceCard4Icon = "📊",
        serviceCard4Title = "Strategic Business Development & Consulting",
        serviceCard4Body = "Niche selection, intelligent pricing models, and service offerings that maximize automation ROI.",
        serviceCard4Image = "",
        serviceCardBg = "",
        serviceCardBorder = "",
        serviceCardTitleFontSize = 1.2,
        serviceCardBodyFontSize = 0.92,
        serviceLearnMoreText = "Learn More →",
        serviceLearnMoreColor = "",

        // === SPEED CALLOUT ===
        showSpeedCallout = true,
        speedBg = "",
        speedNumber = "30 Minutes.",
        speedNumberColor = "",
        speedNumberFontSize = "clamp(3rem, 7vw, 6rem)",
        speedLabel = "From Zero to Live Deployment.",
        speedLabelColor = "",
        speedLabelFontSize = "clamp(1.5rem, 3vw, 2.5rem)",
        speedDesc = "That's how long it took me to design, build, and launch a fully functional AI chatbot for a clinic manager.",
        speedDescColor = "",
        speedSub = "I don't just plan. I execute. Speed without compromise is the standard.",
        speedSubColor = "",

        // === METRICS ===
        showMetrics = true,
        metricsBg = "",
        metric1Value = 8,
        metric1Suffix = "+",
        metric1Label = "Custom Web Platforms Delivered",
        metric1Color = "",
        metric2Text = "24/7",
        metric2Label = "AI Systems Running Live",
        metric2Color = "",
        metric3Text = "0",
        metric3Label = "Dropped Leads Guaranteed",
        metric3Color = "",
        metric4Value = 3,
        metric4Suffix = "+",
        metric4Label = "Videos Published Daily",
        metric4Color = "",
        metricLabelColor = "",
        metricNumberFontSize = "clamp(2rem, 4vw, 3rem)",
        metricLabelFontSize = 0.9,

        // === PORTFOLIO PREVIEW ===
        showPortfolioPreview = true,
        portfolioPreviewBg = "",
        portfolioPreviewLabel = "Selected Work",
        portfolioPreviewTitle = "Projects That Speak for Themselves",
        portfolio1Tag = "Healthcare",
        portfolio1Title = "Classic Clinic",
        portfolio1Desc = "Full healthcare web design, AI automation, and digital infrastructure.",
        portfolio1Image = "",
        portfolio2Tag = "Healthcare",
        portfolio2Title = "Dental Clinic of Compton",
        portfolio2Desc = "Complete dental practice website with service pricing architecture.",
        portfolio2Image = "",
        portfolio3Tag = "Real Estate",
        portfolio3Title = "Dream Realty LA",
        portfolio3Desc = "Luxury real estate digital storefront and lead generation platform.",
        portfolio3Image = "",
        portfolio4Tag = "AI Automation",
        portfolio4Title = "30-Minute Chatbot Deployment",
        portfolio4Desc = "Rapid-build AI chatbot for clinic operations — live in under 30 minutes.",
        portfolio4Image = "",
        portfolioCardBg = "",
        portfolioCardBorder = "",
        portfolioTagBg = "",
        portfolioTagColor = "",
        portfolioTagBorder = "",
        portfolioTagFontSize = 0.7,
        portfolioTitleFontSize = 1.15,
        portfolioDescFontSize = 0.875,
        portfolioViewAllText = "View All Projects →",

        // === PROCESS ===
        showProcess = true,
        processBg = "",
        processLabel = "The Process",
        processTitle = "From Strategy to Deployment",
        processSubtitle = "Here's how I take you from zero to fully automated.",
        process1Num = "01",
        process1Title = "Discovery & Audit",
        process1Body = "I analyze your current operations, tech stack, and pain points. I identify exactly where time, money, and leads are being lost.",
        process2Num = "02",
        process2Title = "Architecture & Strategy",
        process2Body = "I map out the complete system — every automation flow, every page structure, every integration point — before a single line is built.",
        process3Num = "03",
        process3Title = "Rapid Build & Deploy",
        process3Body = "I execute fast. Systems are built, tested, and deployed on aggressive timelines without sacrificing precision or quality.",
        process4Num = "04",
        process4Title = "Optimization & Scale",
        process4Body = "Post-launch, I monitor performance, refine workflows, and expand system capabilities as your business grows.",
        processStepBg = "",
        processStepBorder = "",
        processStepNumberSize = 64,
        processStepTitleFontSize = 1.1,
        processStepBodyFontSize = 0.875,

        // === TESTIMONIALS ===
        showTestimonials = true,
        testimonialsBg = "",
        testimonialsLabel = "What Clients Say",
        testimonialsTitle = "Trusted by Industry Leaders",
        testimonial1Text = '"David completely transformed our clinic\'s operations. The AI chatbot handles patient inquiries 24/7 and our no-show rate dropped by 40%. Absolutely game-changing."',
        testimonial1Name = "Dr. Maria Rodriguez",
        testimonial1Role = "Director, Classic Clinic",
        testimonial1Initials = "MR",
        testimonial1Image = "",
        testimonial1Stars = 5,
        testimonial2Text = '"Our new website generates more qualified leads in a week than our old one did in a month. David builds revenue machines."',
        testimonial2Name = "Toma Barseghian",
        testimonial2Role = "Luxury Real Estate Agent, Dream Realty LA",
        testimonial2Initials = "TB",
        testimonial2Image = "",
        testimonial2Stars = 5,
        testimonial3Text = '"The speed was unbelievable. David had our AI system running the same day. Our front desk can finally focus on patients instead of phone tag."',
        testimonial3Name = "Dr. James Chen",
        testimonial3Role = "Owner, Dental Clinic of Compton",
        testimonial3Initials = "JC",
        testimonial3Image = "",
        testimonial3Stars = 5,
        testimonialCardBg = "",
        testimonialCardBorder = "",
        testimonialTextFontSize = 0.95,
        testimonialTextColor = "",
        testimonialNameFontSize = 0.9,
        testimonialRoleFontSize = 0.8,
        testimonialAvatarBg = "",
        testimonialAvatarBorder = "",
        testimonialAvatarSize = 40,

        // === FINAL CTA ===
        showFinalCta = true,
        finalCtaBg = "",
        finalCtaGlowColor = "",
        finalCtaTitle = "Ready to Stop Losing Revenue to Manual Processes?",
        finalCtaTitleColor = "",
        finalCtaTitleFontSize = "clamp(2rem, 4vw, 3rem)",
        finalCtaBody = "Let's build the system that runs your business 24/7. Book a free strategy call and I'll show you exactly where automation will have the highest impact.",
        finalCtaBodyColor = "",
        finalCtaButton = "Book Your Strategy Call",
        finalCtaButtonBg = "",
        finalCtaButtonColor = "#FFFFFF",
        finalCtaButtonFontSize = 0.95,
        finalCtaEmail = "david@aiaa.agency",
        finalCtaEmailLabel = "Or email me directly →",
        showFinalCtaEmail = true,

        // === SERVICES PAGE ===
        servicesPageTitle = "Services Built to Automate, Convert, and Scale",
        servicesPageSubtitle = "Every system I build is reverse-engineered from one question: how does this directly increase your revenue or eliminate your operational overhead?",
        servicesPageTitleFontSize = "clamp(2.5rem, 5vw, 4rem)",

        sd1Title = "AI Automation & Systems Engineering",
        sd1Icon = "🤖",
        sd1Desc = "I design and deploy intelligent automation systems that replace manual operational tasks with precision AI workflows. Zero downtime, zero human error, zero dropped leads.",
        sd1Sub1Title = "Smart Front Desk Architecture",
        sd1Sub1Body = "Custom-built triage and retention bots for healthcare facilities. Powered by Make.com and OpenAI, available 24/7/365.",
        sd1Sub2Title = "Inbound Workflow Optimization",
        sd1Sub2Body = "Every inbound message is captured, categorized, and responded to instantly. Zero leads fall through the cracks.",
        sd1Sub3Title = "No-Show Prevention Systems",
        sd1Sub3Body = "Automated multi-touchpoint communication sequences that drastically reduce no-show and cancellation rates.",
        sd1Sub4Title = "Reputation Management Engineering",
        sd1Sub4Body = "Automated post-visit review solicitation workflows that systematically build your online reputation.",
        sd1Sub5Title = "Rapid Deployment Capability",
        sd1Sub5Body = "A fully functional chatbot in 30 minutes. Complex enterprise automations in days, not weeks.",

        sd2Title = "Custom Web Design & Digital Infrastructure",
        sd2Icon = "🌐",
        sd2Desc = "High-converting websites that generate revenue. Every element strategically placed to convert visitors into booked appointments.",
        sd2Sub1Title = "Healthcare Web Design",
        sd2Sub1Body = "Complete digital platforms for dental practices and healthcare facilities — wireframing through deployment.",
        sd2Sub2Title = "Real Estate Digital Storefronts",
        sd2Sub2Body = "High-end websites for luxury real estate professionals with property showcases and lead capture.",
        sd2Sub3Title = "Conversion-Optimized Architecture",
        sd2Sub3Body = "Every site engineered for action with frictionless contact pathways and strategic user flows.",

        sd3Title = "High-Volume Content Strategy & Digital Media",
        sd3Icon = "🎬",
        sd3Desc = "Scalable content pipelines built for rapid audience growth and direct monetization.",
        sd3Sub1Title = "AI-Powered Content Generation",
        sd3Sub1Body = "Advanced AI tools for multimedia content structured for platform growth and revenue generation.",
        sd3Sub2Title = "High-Frequency Publishing",
        sd3Sub2Body = "Aggressive content distribution: three high-quality videos per day for rapid audience acquisition.",
        sd3Sub3Title = "Synthetic Audio & Music Production",
        sd3Sub3Body = "Custom AI-generated music tracks that elevate production quality and increase engagement.",

        sd4Title = "Strategic Business Development & Consulting",
        sd4Icon = "📊",
        sd4Desc = "Growth strategy architecture — from niche selection to pricing frameworks to scalable operational design.",
        sd4Sub1Title = "Niche Strategy & Market Positioning",
        sd4Sub1Body = "Identifying high-value niches where automation delivers the highest ROI.",
        sd4Sub2Title = "Scalable Pricing Model Development",
        sd4Sub2Body = "Pricing frameworks that reflect the operational value and transformation you provide.",
        sd4Sub3Title = "Agency Architecture & Operational Design",
        sd4Sub3Body = "Consulting on service refinement, workflow design, and scalable business model architecture.",

        servicesCtaTitle = "Know What You Need? Let's Talk.",
        servicesCtaBody = "Book your strategy call and I'll show you the exact systems for highest impact.",
        servicesCtaButton = "Schedule a Call →",

        // === ABOUT PAGE ===
        aboutPageTitle = "About David",
        aboutPageSubtitle = "Engineer. Strategist. Builder.",
        aboutPageTitleFontSize = "clamp(2.5rem, 5vw, 4rem)",
        aboutPhoto = "",
        aboutPhotoPlaceholderIcon = "👤",
        aboutPhotoPlaceholderText = "David — Founder, AIAA",
        aboutPhotoAspectRatio = "3/4",
        aboutPhotoBorderRadius = 12,
        aboutBio1 = "I'm David — an AI systems engineer, web architect, and digital strategist building the operational backbone for high-value service businesses.",
        aboutBio2 = "I started with a simple observation: the businesses that generate the most revenue are often running on the most outdated, manual, and fragile operational systems. Every one of these failures has a direct cost, and every one is solvable with the right automation.",
        aboutBio3 = "So I build the solutions.",
        aboutBio4 = "I design AI-powered systems using Make.com and OpenAI that handle front desk triage, lead capture, appointment reminders, and review generation — all running autonomously. I build high-converting websites engineered to turn visitors into booked appointments. And I architect content pipelines that scale at a pace manual effort cannot match.",
        aboutBio5 = "I'm not a freelancer assembling templates. I'm an engineer building infrastructure. Every system I deliver generates measurable, compounding returns.",
        aboutBioFontSize = 1,
        aboutBio1FontSize = 1.1,

        principlesLabel = "Core Principles",
        principlesTitle = "How I Operate",
        principle1Num = "01",
        principle1Title = '"Speed Is a Feature."',
        principle1Body = "Full chatbot in 30 minutes. Complete website in days. Speed without quality compromise is a competitive advantage.",
        principle2Num = "02",
        principle2Title = '"Every System Must Pay for Itself."',
        principle2Body = "Every automation is designed with clear revenue impact — recovered leads, reduced no-shows, or increased bookings.",
        principle3Num = "03",
        principle3Title = '"Build Once. Run Forever."',
        principle3Body = "Permanent operational infrastructure that runs autonomously, scales with your business, and compounds in value.",
        principleCardBg = "",
        principleNumColor = "",
        principleNumFontSize = 2.5,
        principleTitleFontSize = 1.15,
        principleBodyFontSize = 0.9,

        focusLabel = "Current Focus",
        focusTitle = "What I'm Building Right Now",
        focus1 = "Scaling AIAA — refining service offerings and onboarding high-value clients in healthcare and real estate.",
        focus2 = "Deepening the AI content pipeline — publishing three videos daily and expanding into new formats.",
        focus3 = "Pushing the technical frontier — integrating more advanced AI capabilities into client systems.",
        focusItemBg = "",
        focusNumColor = "",

        aboutCtaTitle = "Let's Work Together",
        aboutCtaBody = "If you're running a high-value service business and you're ready to automate, convert, and scale — let's talk.",
        aboutCtaButton = "Book a Strategy Call →",

        // === CONTACT PAGE ===
        contactPageTitle = "Let's Build Something",
        contactPageSubtitle = "Whether you need an AI system, a new website, or a complete digital overhaul — it starts with a conversation.",
        contactPageTitleFontSize = "clamp(2.5rem, 5vw, 4rem)",
        bookingTitle = "Book a Free Strategy Call",
        bookingDesc = "Pick a time that works. I'll audit your current setup, identify the highest-impact opportunities, and give you a clear roadmap — no obligation.",
        bookingPlaceholderIcon = "📅",
        bookingPlaceholderText = "Calendly booking widget embeds here",
        bookingLinkText = "Open Scheduling Page →",
        calendlyUrl = "https://calendly.com",
        bookingCardBg = "",

        formTitle = "Send a Message",
        formNameLabel = "Name *",
        formNamePlaceholder = "Your full name",
        formEmailLabel = "Email *",
        formEmailPlaceholder = "your@email.com",
        formPhoneLabel = "Phone (optional)",
        formPhonePlaceholder = "(555) 123-4567",
        formBusinessLabel = "Business Type",
        formBusinessDefault = "Select your industry",
        formBusinessOption1 = "Healthcare",
        formBusinessOption2 = "Real Estate",
        formBusinessOption3 = "Other",
        formMessageLabel = "What do you need help with? *",
        formMessagePlaceholder = "Tell me about your project...",
        formSubmitText = "Send Message",
        formSuccessIcon = "✅",
        formSuccessTitle = "Message Sent!",
        formSuccessBody = "I'll get back to you within 24 hours.",
        formCardBg = "",
        formInputBg = "",
        formInputBorder = "",
        formInputColor = "",
        formSubmitBg = "",
        formSubmitColor = "#FFFFFF",

        contactEmail = "david@aiaa.agency",
        contactEmailIcon = "📧",
        contactEmailLabel = "Email",
        contactLocation = "Los Angeles, CA — Serving clients remotely nationwide",
        contactLocationIcon = "📍",
        contactLocationLabel = "Location",
        contactResponse = "Typically within 2-4 hours during business days",
        contactResponseIcon = "⚡",
        contactResponseLabel = "Response Time",
        contactCardBg = "",

        // === BLOG PAGE ===
        blogPageTitle = "Insights",
        blogPageSubtitle = "Thoughts on AI automation, digital strategy, and building systems that scale.",
        blogPageTitleFontSize = "clamp(2.5rem, 5vw, 4rem)",

        blog1Icon = "🏥",
        blog1Date = "January 15, 2025",
        blog1Title = "Why Your Dental Practice Is Losing $10K/Month to No-Shows",
        blog1Excerpt = "The average dental practice loses between $8,000 and $15,000 per month to no-shows. Here's the automated system that eliminates this.",
        blog1Image = "",
        blog2Icon = "🤖",
        blog2Date = "January 28, 2025",
        blog2Title = "I Built a Fully Functional AI Chatbot in 30 Minutes — Here's How",
        blog2Excerpt = "A step-by-step breakdown of how I deployed a production-ready AI chatbot in half an hour flat.",
        blog2Image = "",
        blog3Icon = "⭐",
        blog3Date = "February 10, 2025",
        blog3Title = "The Automated Review System That Tripled a Clinic's Google Rating",
        blog3Excerpt = "How a simple automation workflow turned a 3.2-star clinic into a 4.8-star powerhouse.",
        blog3Image = "",
        blog4Icon = "🏡",
        blog4Date = "February 24, 2025",
        blog4Title = "Why High-End Real Estate Agents Need More Than a Pretty Website",
        blog4Excerpt = "A beautiful website means nothing if it doesn't convert. Here's what actually works.",
        blog4Image = "",
        blog5Icon = "🎬",
        blog5Date = "March 8, 2025",
        blog5Title = "3 Videos a Day: How I Built a Scalable AI Content Pipeline",
        blog5Excerpt = "The exact system I use to produce and publish three high-quality videos daily.",
        blog5Image = "",
        blogCardBg = "",
        blogCardBorder = "",
        blogImageBg = "",
        blogDateColor = "",
        blogTitleFontSize = 1.15,
        blogExcerptFontSize = 0.875,
        blogReadMoreText = "Read More →",
        blogReadMoreColor = "",

        // === FOOTER ===
        footerBg = "",
        footerBorderColor = "",
        footerCopy = "© 2025 David — AI Automation Agency. All rights reserved.",
        footerCopyColor = "",
        footerLinkColor = "",
        showFooterLinks = true,
        showFooterSocial = true,
        socialLinkedinUrl = "https://linkedin.com",
        socialLinkedinLabel = "in",
        socialYoutubeUrl = "https://youtube.com",
        socialYoutubeLabel = "▶",
        socialTwitterUrl = "https://twitter.com",
        socialTwitterLabel = "𝕏",
        socialGithubUrl = "",
        socialGithubLabel = "GH",
        socialIconBg = "transparent",
        socialIconBorder = "",
        socialIconColor = "",
        socialIconSize = 36,

        // === PORTFOLIO PAGE FULL ===
        portfolioPageTitle = "The Work",
        portfolioPageSubtitle = "Real systems. Real clients. Real results.",
        portfolioFilterAll = "All Projects",
        portfolioFilterHealthcare = "Healthcare",
        portfolioFilterRealEstate = "Real Estate",
        portfolioFilterAI = "AI Automation",
        portfolioFilterContent = "Content",
        filterBtnBg = "transparent",
        filterBtnActiveBg = "",
        filterBtnBorder = "",
        filterBtnColor = "",
        filterBtnActiveColor = "#FFFFFF",

        pf1Title = "Classic Clinic",
        pf1Desc = "Full healthcare web design, AI-powered patient triage, and automated scheduling.",
        pf1Tags = "healthcare",
        pf1Image = "",
        pf2Title = "Dental Clinic of Compton",
        pf2Desc = "Custom dental practice website with comprehensive service pricing architecture.",
        pf2Tags = "healthcare",
        pf2Image = "",
        pf3Title = "Toma Barseghian — Dream Realty LA",
        pf3Desc = "Luxury real estate web platform with property showcase and lead capture.",
        pf3Tags = "real-estate",
        pf3Image = "",
        pf4Title = "Frank Lopez — Real Estate",
        pf4Desc = "Professional agent website with listing integration and lead qualification.",
        pf4Tags = "real-estate",
        pf4Image = "",
        pf5Title = "Alina Muradyan — Real Estate",
        pf5Desc = "Premium real estate web presence with brand-first design.",
        pf5Tags = "real-estate",
        pf5Image = "",
        pf6Title = "30-Minute Chatbot Deployment",
        pf6Desc = "Rapid-build AI chatbot — live in under 30 minutes.",
        pf6Tags = "ai",
        pf6Image = "",
        pf7Title = "YouTube Content Pipeline",
        pf7Desc = "AI-powered video production — 3 videos per day, fully automated.",
        pf7Tags = "content",
        pf7Image = "",
        pf8Title = "Automated No-Show Prevention",
        pf8Desc = "Multi-touchpoint reminder system reducing no-shows by 40%.",
        pf8Tags = "ai,healthcare",
        pf8Image = "",

        portfolioCtaTitle = "Want Results Like These?",
        portfolioCtaBody = "Let's discuss your project and map out the highest-impact system.",
        portfolioCtaButton = "Start Your Project →",

        // === CASE STUDY MODAL ===
        csModalBg = "rgba(0,0,0,0.85)",
        csContentBg = "",
        csChallengeLabel = "The Challenge",
        csSolutionLabel = "The Solution",
        csOutcomeLabel = "The Outcome",
        csIndustryLabel = "Industry",
        csScopeLabel = "Scope",
        csCloseIcon = "✕",

        // Full case study content
        cs1Tag = "Healthcare • Web Design • AI Automation",
        cs1Title = "Classic Clinic",
        cs1Industry = "Healthcare — Multi-Specialty",
        cs1Scope = "Full website design, AI chatbot, automated scheduling, review system",
        cs1Challenge = "Classic Clinic was operating with an outdated website that generated zero online leads. Their front desk was overwhelmed with phone calls, resulting in missed inquiries and a growing no-show problem. Their Google rating sat at 3.4 stars with only 12 reviews.",
        cs1Solution = "I designed and built a complete digital platform from scratch — a conversion-optimized website with strategic service page architecture. Simultaneously, I deployed an AI-powered chatbot using Make.com and OpenAI. I also implemented an automated no-show prevention system and post-visit review solicitation workflow.",
        cs1Outcome = "Within 60 days: 65% increase in online bookings. AI chatbot handles 47 inquiries/day. No-show rates dropped 38%. Google rating: 3.4 → 4.7 with 85+ new reviews.",

        cs2Tag = "Healthcare • Web Design",
        cs2Title = "Dental Clinic of Compton",
        cs2Industry = "Healthcare — Dental",
        cs2Scope = "Custom dental website, pricing architecture, review automation",
        cs2Challenge = "No web presence. Patients found them only through word-of-mouth. No review system, inconsistent pricing.",
        cs2Solution = "Built comprehensive dental website with clear service categorization, transparent pricing, patient-first architecture, streamlined booking, and automated review solicitation.",
        cs2Outcome = "Zero to fully operational in 30 days. 23 new patient bookings via website. Reviews: 4 → 31 in six weeks, 4.9 average.",

        cs3Tag = "Real Estate • Web Design",
        cs3Title = "Toma Barseghian — Dream Realty LA",
        cs3Industry = "Real Estate — Luxury Residential",
        cs3Scope = "Luxury web platform, agent branding, lead capture",
        cs3Challenge = "Generic template page on brokerage site. No premium positioning, no inbound leads.",
        cs3Solution = "Custom luxury platform with premium design, property showcases, testimonials, neighborhood guides, and minimal-friction lead capture with instant response automation.",
        cs3Outcome = "Lead submissions up 300%. Time-on-site: 45 seconds → 3+ minutes.",

        cs4Tag = "Real Estate • Web Design",
        cs4Title = "Frank Lopez — Real Estate",
        cs4Industry = "Real Estate — Residential",
        cs4Scope = "Agent website, listing integration, lead qualification",
        cs4Challenge = "Entirely referral-based marketing. No digital infrastructure.",
        cs4Solution = "Professional website with listing integration, neighborhood spotlights, and intelligent lead qualification questionnaires.",
        cs4Outcome = "Primary lead generation tool within 45 days. Saves 8 hours/week on unqualified prospects.",

        cs5Tag = "Real Estate • Web Design",
        cs5Title = "Alina Muradyan — Real Estate",
        cs5Industry = "Real Estate — Residential",
        cs5Scope = "Premium web presence, brand design, engagement system",
        cs5Challenge = "Strong offline reputation but zero digital presence.",
        cs5Solution = "Brand-first website with personal story, client testimonials, property portfolio, and automated response sequences.",
        cs5Outcome = "12 monthly web inquiries with 40% conversion to consultation — above industry average.",

        cs6Tag = "AI Automation",
        cs6Title = "30-Minute Chatbot Deployment",
        cs6Industry = "Healthcare — Clinic Management",
        cs6Scope = "AI chatbot design, configuration, testing, live deployment",
        cs6Challenge = "Needed after-hours chatbot. Previous quotes: 2-4 weeks, $5,000+.",
        cs6Solution = "Make.com + OpenAI chatbot handling FAQs, appointment capture, and escalation. Built in 30 minutes.",
        cs6Outcome = "Live in 30 minutes. 28 daily after-hours inquiries. Zero missed opportunities. Fraction of competitive cost.",

        cs7Tag = "Content Strategy • AI",
        cs7Title = "YouTube Content Pipeline",
        cs7Industry = "Digital Media",
        cs7Scope = "AI video production, automated publishing",
        cs7Challenge = "Manual production cannot sustain 3 videos/day with quality.",
        cs7Solution = "Complete AI pipeline: topic research, script generation, synthetic voiceover, dynamic visuals, AI music, scheduled distribution.",
        cs7Outcome = "3 videos/day, 90+/month. Exponential channel growth aligned with YouTube algorithm.",

        cs8Tag = "AI Automation • Healthcare",
        cs8Title = "Automated No-Show Prevention",
        cs8Industry = "Healthcare — Dental & Medical",
        cs8Scope = "Multi-touchpoint reminders, rebooking automation",
        cs8Challenge = "28% no-show rate = ~$12,000/month lost. Staff spending 2-3 hours daily on manual calls.",
        cs8Solution = "SMS 72h before, email 24h before, SMS 2h before. Non-confirmations trigger rebooking. Real-time analytics.",
        cs8Outcome = "No-shows: 28% → 11%. ~$8,500/month recovered. Staff call time eliminated.",

        // === BLOG FULL ARTICLES ===
        blogArticle1Body = "Every dental practice deals with no-shows. Most accept it as a cost of doing business. The average practice loses $8,000-$15,000 per month.\n\nThe system I build uses multi-touchpoint automated communication: SMS 72 hours out, email 24 hours out, final SMS 2 hours before. Non-confirmations trigger rebooking.\n\nResult: No-show rates drop from 25-30% to 8-12% within the first month.",
        blogArticle2Body = "A clinic manager needed an AI chatbot. Previous vendors quoted 2-4 weeks and $5,000+. I had it live in 30 minutes.\n\nThe stack: Make.com for automation, OpenAI for NLU. No custom code.\n\nStep 1: Define use case (5 min). Step 2: Build flow (15 min). Step 3: Test and deploy (10 min).\n\nNow handles 28 after-hours inquiries daily.",
        blogArticle3Body = "A clinic had 3.2 stars with 8 reviews. Six weeks later: 4.8 stars with 39 reviews.\n\nThe system triggers 2 hours post-appointment: personalized SMS with direct Google review link. 24-hour email follow-up.\n\nResult: 31 new reviews, 4.9 average. Top 3 Google Maps for their specialty.",
        blogArticle4Body = "Every luxury agent thinks they need a 'beautiful' website. They need a revenue machine.\n\nConversion-first architecture: Above-fold contact, featured listings, authority section, neighborhood content, instant-response lead capture.\n\nMy platforms average 4.2% lead form rate vs 1.5% industry average.",
        blogArticle5Body = "3 videos/day sounds impossible manually. My AI pipeline:\n\nTopic Generation: AI analyzes trends, generates 21 weekly topics.\n\nProduction: AI scripts, synthetic voiceover, dynamic visuals, AI music.\n\nPublishing: Auto-upload with optimized metadata, scheduled for peak engagement.\n\nResult: 90+ videos/month, exponential growth.",

        // === BLOG MODAL ===
        blogModalBg = "rgba(0,0,0,0.85)",
        blogModalContentBg = "",
        blogModalTitleFontSize = 1.75,
    } = props

    // ==========================================
    // STATE
    // ==========================================
    const [currentPage, setCurrentPage] = useState("home")
    const [mobileMenuOpen, setMobileMenuOpen] = useState(false)
    const [activeFilter, setActiveFilter] = useState("all")
    const [activeCaseStudy, setActiveCaseStudy] = useState<CaseStudy | null>(
        null
    )
    const [activeBlog, setActiveBlog] = useState<BlogArticle | null>(null)
    const [formSubmitted, setFormSubmitted] = useState(false)

    // ==========================================
    // DERIVED COLORS
    // ==========================================
    const ac = accentColor
    const acGlow = ac + "26"
    const acBorder = ac + "33"
    const resolveColor = (custom: string, fallback: string) =>
        custom && custom.trim() !== "" ? custom : fallback

    // ==========================================
    // CASE STUDIES DATA
    // ==========================================
    const caseStudies: Record<string, CaseStudy> = {
        "1": {
            tag: cs1Tag,
            title: cs1Title,
            industry: cs1Industry,
            scope: cs1Scope,
            challenge: cs1Challenge,
            solution: cs1Solution,
            outcome: cs1Outcome,
        },
        "2": {
            tag: cs2Tag,
            title: cs2Title,
            industry: cs2Industry,
            scope: cs2Scope,
            challenge: cs2Challenge,
            solution: cs2Solution,
            outcome: cs2Outcome,
        },
        "3": {
            tag: cs3Tag,
            title: cs3Title,
            industry: cs3Industry,
            scope: cs3Scope,
            challenge: cs3Challenge,
            solution: cs3Solution,
            outcome: cs3Outcome,
        },
        "4": {
            tag: cs4Tag,
            title: cs4Title,
            industry: cs4Industry,
            scope: cs4Scope,
            challenge: cs4Challenge,
            solution: cs4Solution,
            outcome: cs4Outcome,
        },
        "5": {
            tag: cs5Tag,
            title: cs5Title,
            industry: cs5Industry,
            scope: cs5Scope,
            challenge: cs5Challenge,
            solution: cs5Solution,
            outcome: cs5Outcome,
        },
        "6": {
            tag: cs6Tag,
            title: cs6Title,
            industry: cs6Industry,
            scope: cs6Scope,
            challenge: cs6Challenge,
            solution: cs6Solution,
            outcome: cs6Outcome,
        },
        "7": {
            tag: cs7Tag,
            title: cs7Title,
            industry: cs7Industry,
            scope: cs7Scope,
            challenge: cs7Challenge,
            solution: cs7Solution,
            outcome: cs7Outcome,
        },
        "8": {
            tag: cs8Tag,
            title: cs8Title,
            industry: cs8Industry,
            scope: cs8Scope,
            challenge: cs8Challenge,
            solution: cs8Solution,
            outcome: cs8Outcome,
        },
    }

    const blogArticles: Record<string, BlogArticle> = {
        "1": {
            title: blog1Title,
            date: blog1Date + " • 6 min read",
            body: blogArticle1Body,
        },
        "2": {
            title: blog2Title,
            date: blog2Date + " • 5 min read",
            body: blogArticle2Body,
        },
        "3": {
            title: blog3Title,
            date: blog3Date + " • 4 min read",
            body: blogArticle3Body,
        },
        "4": {
            title: blog4Title,
            date: blog4Date + " • 5 min read",
            body: blogArticle4Body,
        },
        "5": {
            title: blog5Title,
            date: blog5Date + " • 6 min read",
            body: blogArticle5Body,
        },
    }

    // ==========================================
    // NAVIGATION
    // ==========================================
    const navigate = useCallback((page: string) => {
        setCurrentPage(page)
        setMobileMenuOpen(false)
        setFormSubmitted(false)
        window.scrollTo({ top: 0, behavior: "instant" as ScrollBehavior })
    }, [])

    const navItems = [
        { id: "home", label: navLinkHome },
        { id: "services", label: navLinkServices },
        { id: "portfolio", label: navLinkPortfolio },
        { id: "about", label: navLinkAbout },
        { id: "blog", label: navLinkBlog },
        { id: "contact", label: navLinkContact },
    ]

    // ==========================================
    // REUSABLE COMPONENTS
    // ==========================================
    const Container = ({
        children,
        style = {},
    }: {
        children: React.ReactNode
        style?: React.CSSProperties
    }) => (
        <div
            style={{
                maxWidth: containerMaxWidth,
                margin: "0 auto",
                padding: "0 24px",
                width: "100%",
                boxSizing: "border-box" as const,
                ...style,
            }}
        >
            {children}
        </div>
    )

    const SectionLabel = ({ children }: { children: React.ReactNode }) => (
        <span
            style={{
                fontFamily: bodyFont,
                fontSize: `${trustLabelFontSize}rem`,
                fontWeight: 600,
                letterSpacing: 3,
                textTransform: "uppercase" as const,
                color: resolveColor(problemsLabelColor, ac),
                marginBottom: 16,
                display: "block",
            }}
        >
            {children}
        </span>
    )

    const SectionTitle = ({
        children,
        style = {},
    }: {
        children: React.ReactNode
        style?: React.CSSProperties
    }) => (
        <h2
            style={{
                fontFamily: headingFont,
                fontSize: problemsTitleFontSize,
                fontWeight: headingWeight,
                lineHeight: 1.15,
                marginBottom: 16,
                letterSpacing: -1,
                color: resolveColor(problemsTitleColor, textPrimary),
                ...style,
            }}
        >
            {children}
        </h2>
    )

    const Card = ({
        children,
        style = {},
        onClick,
    }: {
        children: React.ReactNode
        style?: React.CSSProperties
        onClick?: () => void
    }) => (
        <motion.div
            whileHover={{ y: -4, borderColor: borderLightColor }}
            style={{
                padding: "40px 32px",
                background: resolveColor(problemCardBg, bgCard),
                border: `1px solid ${resolveColor(problemCardBorder, borderColor)}`,
                borderRadius: cardBorderRadius,
                cursor: onClick ? "pointer" : "default",
                transition: "all 0.4s ease",
                ...style,
            }}
            onClick={onClick}
        >
            {children}
        </motion.div>
    )

    const IconBox = ({
        children,
        size = problemIconSize,
    }: {
        children: React.ReactNode
        size?: number
    }) => (
        <div
            style={{
                width: size,
                height: size,
                background: resolveColor(problemIconBg, acGlow),
                border: `1px solid ${resolveColor(problemIconBorder, acBorder)}`,
                borderRadius: 8,
                display: "flex",
                alignItems: "center",
                justifyContent: "center",
                fontSize: size * 0.27,
                marginBottom: 24,
            }}
        >
            {children}
        </div>
    )

    const BtnPrimary = ({
        children,
        onClick,
        style = {},
    }: {
        children: React.ReactNode
        onClick?: () => void
        style?: React.CSSProperties
    }) => (
        <motion.button
            whileHover={{ y: -2, boxShadow: `0 8px 30px ${ac}4D` }}
            onClick={onClick}
            style={{
                display: "inline-flex",
                alignItems: "center",
                gap: 8,
                padding: "14px 32px",
                background: resolveColor(finalCtaButtonBg, ac),
                color: finalCtaButtonColor,
                fontWeight: 600,
                fontSize: `${finalCtaButtonFontSize}rem`,
                borderRadius: buttonBorderRadius,
                border: `2px solid ${resolveColor(finalCtaButtonBg, ac)}`,
                cursor: "pointer",
                fontFamily: bodyFont,
                textDecoration: "none",
                ...style,
            }}
        >
            {children}
        </motion.button>
    )

    const BtnSecondary = ({
        children,
        onClick,
        style = {},
    }: {
        children: React.ReactNode
        onClick?: () => void
        style?: React.CSSProperties
    }) => (
        <motion.button
            whileHover={{ y: -2, borderColor: textPrimary }}
            onClick={onClick}
            style={{
                display: "inline-flex",
                alignItems: "center",
                gap: 8,
                padding: "14px 32px",
                background: "transparent",
                color: textPrimary,
                fontWeight: 600,
                fontSize: "0.95rem",
                borderRadius: buttonBorderRadius,
                border: `2px solid ${resolveColor(heroCta2BorderColor, borderLightColor)}`,
                cursor: "pointer",
                fontFamily: bodyFont,
                ...style,
            }}
        >
            {children}
        </motion.button>
    )

    const PortfolioImageSlot = ({
        src,
        variant,
        height = 240,
    }: {
        src?: string
        variant: number
        height?: number
    }) => (
        <div style={{ width: "100%", height }}>
            {src && src.trim() !== "" ? (
                <img
                    src={src}
                    alt=""
                    style={{
                        width: "100%",
                        height: "100%",
                        objectFit: "cover",
                    }}
                />
            ) : (
                <MockUI
                    variant={variant}
                    accentColor={ac}
                    bgColor={bgPrimary}
                    borderColor={borderLightColor}
                />
            )}
        </div>
    )

    const Stars = ({ count }: { count: number }) => (
        <div
            style={{
                color: starColor,
                fontSize: "0.9rem",
                marginBottom: 16,
                letterSpacing: 2,
            }}
        >
            {"★".repeat(count)}
            {"☆".repeat(5 - count)}
        </div>
    )

    const Avatar = ({
        image,
        initials,
    }: {
        image?: string
        initials: string
    }) =>
        image && image.trim() !== "" ? (
            <img
                src={image}
                alt=""
                style={{
                    width: testimonialAvatarSize,
                    height: testimonialAvatarSize,
                    borderRadius: "50%",
                    objectFit: "cover",
                }}
            />
        ) : (
            <div
                style={{
                    width: testimonialAvatarSize,
                    height: testimonialAvatarSize,
                    background: resolveColor(testimonialAvatarBg, acGlow),
                    border: `1px solid ${resolveColor(testimonialAvatarBorder, ac + "4D")}`,
                    borderRadius: "50%",
                    display: "flex",
                    alignItems: "center",
                    justifyContent: "center",
                    fontWeight: 700,
                    fontSize: "0.85rem",
                    color: ac,
                }}
            >
                {initials}
            </div>
        )

    // ==========================================
    // SERVICE DETAIL BLOCK
    // ==========================================
    const ServiceBlock = ({
        icon,
        title,
        desc,
        subServices,
        bg,
    }: {
        icon: string
        title: string
        desc: string
        subServices: { title: string; body: string }[]
        bg?: string
    }) => (
        <section
            style={{
                padding: "80px 0",
                borderTop: `1px solid ${borderColor}`,
                background: bg || "transparent",
            }}
        >
            <Container>
                <FadeIn>
                    <div
                        style={{
                            display: "flex",
                            alignItems: "flex-start",
                            gap: 24,
                            marginBottom: 48,
                            flexWrap: "wrap" as const,
                        }}
                    >
                        <div
                            style={{
                                width: 56,
                                height: 56,
                                minWidth: 56,
                                background: acGlow,
                                border: `1px solid ${acBorder}`,
                                borderRadius: 8,
                                display: "flex",
                                alignItems: "center",
                                justifyContent: "center",
                                fontSize: "1.5rem",
                            }}
                        >
                            {icon}
                        </div>
                        <div>
                            <h2
                                style={{
                                    fontFamily: headingFont,
                                    fontSize: "1.75rem",
                                    fontWeight: headingWeight,
                                    marginBottom: 8,
                                }}
                            >
                                {title}
                            </h2>
                            <p
                                style={{
                                    fontSize: "1rem",
                                    color: textSecondary,
                                    lineHeight: 1.7,
                                    maxWidth: 700,
                                }}
                            >
                                {desc}
                            </p>
                        </div>
                    </div>
                </FadeIn>
                <div
                    style={{
                        display: "grid",
                        gridTemplateColumns:
                            "repeat(auto-fit, minmax(300px, 1fr))",
                        gap: 24,
                    }}
                >
                    {subServices.map((sub, i) => (
                        <FadeIn key={i} delay={i * 0.1}>
                            <Card style={{ padding: 32 }}>
                                <h4
                                    style={{
                                        fontFamily: headingFont,
                                        fontSize: "1.05rem",
                                        fontWeight: 600,
                                        marginBottom: 10,
                                    }}
                                >
                                    {sub.title}
                                </h4>
                                <p
                                    style={{
                                        fontSize: "0.9rem",
                                        color: textSecondary,
                                        lineHeight: 1.7,
                                    }}
                                >
                                    {sub.body}
                                </p>
                            </Card>
                        </FadeIn>
                    ))}
                </div>
            </Container>
        </section>
    )

    // ==========================================
    // SECTION CTA BLOCK
    // ==========================================
    const CtaSection = ({
        title,
        body,
        button,
        onButtonClick,
    }: {
        title: string
        body: string
        button: string
        onButtonClick: () => void
    }) => (
        <section
            style={{
                padding: `${sectionPadding}px 0`,
                textAlign: "center" as const,
                position: "relative" as const,
                overflow: "hidden",
            }}
        >
            <div
                style={{
                    position: "absolute",
                    top: "50%",
                    left: "50%",
                    transform: "translate(-50%,-50%)",
                    width: 600,
                    height: 600,
                    background: `radial-gradient(circle, ${resolveColor(finalCtaGlowColor, acGlow)} 0%, transparent 70%)`,
                    pointerEvents: "none" as const,
                }}
            />
            <Container style={{ position: "relative", zIndex: 1 }}>
                <FadeIn>
                    <h2
                        style={{
                            fontFamily: headingFont,
                            fontSize: finalCtaTitleFontSize,
                            fontWeight: headingWeight,
                            marginBottom: 16,
                            color: resolveColor(
                                finalCtaTitleColor,
                                textPrimary
                            ),
                        }}
                    >
                        {title}
                    </h2>
                    <p
                        style={{
                            fontSize: "1.1rem",
                            color: resolveColor(
                                finalCtaBodyColor,
                                textSecondary
                            ),
                            maxWidth: 580,
                            margin: "0 auto 40px",
                            lineHeight: 1.7,
                        }}
                    >
                        {body}
                    </p>
                    <BtnPrimary onClick={onButtonClick}>{button}</BtnPrimary>
                </FadeIn>
            </Container>
        </section>
    )

    // ==========================================
    // PAGE: HOME
    // ==========================================
    const renderHome = () => (
        <div style={{ paddingTop: navHeight }}>
            {/* Hero */}
            <section
                style={{
                    minHeight: heroMinHeight,
                    display: "flex",
                    alignItems: "center",
                    position: "relative",
                    overflow: "hidden",
                }}
            >
                {heroBgImage && heroBgImage.trim() !== "" && (
                    <div style={{ position: "absolute", inset: 0, zIndex: 0 }}>
                        <img
                            src={heroBgImage}
                            alt=""
                            style={{
                                width: "100%",
                                height: "100%",
                                objectFit: "cover",
                                opacity: 0.3,
                            }}
                        />
                    </div>
                )}
                {showParticles && (
                    <div style={{ position: "absolute", inset: 0, zIndex: 0 }}>
                        <ParticleCanvas color={ac} />
                    </div>
                )}
                <div
                    style={{
                        position: "absolute",
                        inset: 0,
                        background: `radial-gradient(ellipse at 30% 50%, ${resolveColor(heroGradientColor, ac)}${Math.round(
                            heroGradientOpacity * 255
                        )
                            .toString(16)
                            .padStart(2, "0")} 0%, transparent 60%)`,
                        zIndex: 1,
                    }}
                />
                <Container style={{ position: "relative", zIndex: 2 }}>
                    <div
                        style={{
                            maxWidth: 800,
                            textAlign: heroContentAlign as any,
                        }}
                    >
                        <FadeIn>
                            <div
                                style={{
                                    display: "inline-flex",
                                    alignItems: "center",
                                    gap: 8,
                                    padding: "8px 16px",
                                    background: resolveColor(
                                        heroTagBg,
                                        ac + "1A"
                                    ),
                                    border: `1px solid ${resolveColor(heroTagBorderColor, acBorder)}`,
                                    borderRadius: 50,
                                    fontSize: `${heroTagFontSize}rem`,
                                    fontWeight: 600,
                                    color: resolveColor(heroTagColor, ac),
                                    marginBottom: 24,
                                }}
                            >
                                <motion.span
                                    animate={{ opacity: [1, 0.3, 1] }}
                                    transition={{
                                        duration: 2,
                                        repeat: Infinity,
                                    }}
                                    style={{
                                        width: 6,
                                        height: 6,
                                        background: resolveColor(
                                            heroTagColor,
                                            ac
                                        ),
                                        borderRadius: "50%",
                                        display: "inline-block",
                                    }}
                                />
                                {heroTag}
                            </div>
                        </FadeIn>
                        <FadeIn delay={0.1}>
                            <h1
                                style={{
                                    fontFamily: headingFont,
                                    fontSize: heroHeadlineFontSize,
                                    fontWeight: headingWeight,
                                    lineHeight: heroHeadlineLineHeight,
                                    letterSpacing: heroHeadlineLetterSpacing,
                                    marginBottom: 24,
                                }}
                            >
                                {heroHeadline1}
                                <span
                                    style={{
                                        background: `linear-gradient(135deg, ${resolveColor(heroHighlightGradient1, ac)}, ${resolveColor(heroHighlightGradient2, gradientSecondary)})`,
                                        WebkitBackgroundClip: "text",
                                        WebkitTextFillColor: "transparent",
                                        backgroundClip: "text",
                                    }}
                                >
                                    {heroHighlight}
                                </span>
                                {heroHeadline2}
                            </h1>
                        </FadeIn>
                        <FadeIn delay={0.2}>
                            <p
                                style={{
                                    fontSize: `${heroSubFontSize}rem`,
                                    color: resolveColor(
                                        heroSubColor,
                                        textSecondary
                                    ),
                                    lineHeight: 1.7,
                                    marginBottom: 40,
                                    maxWidth: heroSubMaxWidth,
                                }}
                            >
                                {heroSubheadline}
                            </p>
                        </FadeIn>
                        <FadeIn delay={0.3}>
                            <div
                                style={{
                                    display: "flex",
                                    gap: 16,
                                    flexWrap: "wrap" as const,
                                    justifyContent:
                                        heroContentAlign === "center"
                                            ? "center"
                                            : "flex-start",
                                }}
                            >
                                <BtnPrimary
                                    onClick={() => navigate("portfolio")}
                                    style={{
                                        background: resolveColor(
                                            heroCta1Bg,
                                            ac
                                        ),
                                        borderColor: resolveColor(
                                            heroCta1Bg,
                                            ac
                                        ),
                                        color: heroCta1Color,
                                        fontSize: `${heroCta1FontSize}rem`,
                                    }}
                                >
                                    {heroCta1Text}
                                    {showHeroCta1Icon && (
                                        <span>{heroCta1Icon}</span>
                                    )}
                                </BtnPrimary>
                                <BtnSecondary
                                    onClick={() => navigate("contact")}
                                    style={{
                                        color: resolveColor(
                                            heroCta2Color,
                                            textPrimary
                                        ),
                                        fontSize: `${heroCta2FontSize}rem`,
                                    }}
                                >
                                    {heroCta2Text}
                                </BtnSecondary>
                            </div>
                        </FadeIn>
                    </div>
                </Container>
            </section>

            {/* Trust Strip */}
            {showTrustStrip && (
                <section
                    style={{
                        padding: "48px 0",
                        borderTop: `1px solid ${borderColor}`,
                        borderBottom: `1px solid ${borderColor}`,
                        background: resolveColor(trustBg, bgSecondary),
                    }}
                >
                    <Container>
                        <FadeIn>
                            <div
                                style={{
                                    fontSize: `${trustLabelFontSize}rem`,
                                    fontWeight: 600,
                                    letterSpacing: 3,
                                    textTransform: "uppercase" as const,
                                    color: resolveColor(
                                        trustLabelColor,
                                        textMuted
                                    ),
                                    textAlign: "center" as const,
                                    marginBottom: 24,
                                }}
                            >
                                {trustLabel}
                            </div>
                            <div
                                style={{
                                    display: "flex",
                                    alignItems: "center",
                                    justifyContent: "center",
                                    flexWrap: "wrap" as const,
                                    gap: 40,
                                }}
                            >
                                {[
                                    { name: trust1, logo: trustLogo1 },
                                    { name: trust2, logo: trustLogo2 },
                                    { name: trust3, logo: trustLogo3 },
                                    { name: trust4, logo: trustLogo4 },
                                    { name: trust5, logo: trustLogo5 },
                                    { name: trust6, logo: trustLogo6 },
                                ].map((item, i) => (
                                    <span key={i}>
                                        {item.logo &&
                                        item.logo.trim() !== "" ? (
                                            <img
                                                src={item.logo}
                                                alt={item.name}
                                                style={{
                                                    height: trustLogoHeight,
                                                    objectFit: "contain",
                                                }}
                                            />
                                        ) : (
                                            <span
                                                style={{
                                                    fontFamily: headingFont,
                                                    fontSize: `${trustNameFontSize}rem`,
                                                    fontWeight: 500,
                                                    color: resolveColor(
                                                        trustNameColor,
                                                        textMuted
                                                    ),
                                                    whiteSpace:
                                                        "nowrap" as const,
                                                }}
                                            >
                                                {item.name}
                                            </span>
                                        )}
                                    </span>
                                ))}
                            </div>
                        </FadeIn>
                    </Container>
                </section>
            )}

            {/* Problems */}
            {showProblems && (
                <section
                    style={{
                        padding: `${sectionPadding}px 0`,
                        background: resolveColor(problemsBg, bgPrimary),
                    }}
                >
                    <Container>
                        <FadeIn>
                            <SectionLabel>{problemsLabel}</SectionLabel>
                        </FadeIn>
                        <FadeIn delay={0.1}>
                            <SectionTitle>{problemsTitle}</SectionTitle>
                        </FadeIn>
                        <FadeIn delay={0.2}>
                            <p
                                style={{
                                    fontSize: "1.1rem",
                                    color: resolveColor(
                                        problemsSubColor,
                                        textSecondary
                                    ),
                                    maxWidth: 640,
                                    lineHeight: 1.7,
                                }}
                            >
                                {problemsSubtitle}
                            </p>
                        </FadeIn>
                        <div
                            style={{
                                display: "grid",
                                gridTemplateColumns:
                                    "repeat(auto-fit, minmax(300px, 1fr))",
                                gap: 32,
                                marginTop: 64,
                            }}
                        >
                            {[
                                {
                                    icon: problem1Icon,
                                    title: problem1Title,
                                    body: problem1Body,
                                },
                                {
                                    icon: problem2Icon,
                                    title: problem2Title,
                                    body: problem2Body,
                                },
                                {
                                    icon: problem3Icon,
                                    title: problem3Title,
                                    body: problem3Body,
                                },
                            ].map((item, i) => (
                                <FadeIn key={i} delay={i * 0.1}>
                                    <Card>
                                        <IconBox>{item.icon}</IconBox>
                                        <h3
                                            style={{
                                                fontFamily: headingFont,
                                                fontSize: `${problemTitleFontSize}rem`,
                                                fontWeight: 600,
                                                marginBottom: 12,
                                                lineHeight: 1.3,
                                                color: resolveColor(
                                                    problem1TitleColor,
                                                    textPrimary
                                                ),
                                            }}
                                        >
                                            {item.title}
                                        </h3>
                                        <p
                                            style={{
                                                fontSize: `${problemBodyFontSize}rem`,
                                                color: resolveColor(
                                                    problem1BodyColor,
                                                    textSecondary
                                                ),
                                                lineHeight: 1.7,
                                            }}
                                        >
                                            {item.body}
                                        </p>
                                    </Card>
                                </FadeIn>
                            ))}
                        </div>
                    </Container>
                </section>
            )}

            {/* Services Overview */}
            {showServicesOverview && (
                <section
                    style={{
                        padding: `${sectionPadding}px 0`,
                        background: resolveColor(
                            servicesOverviewBg,
                            bgSecondary
                        ),
                    }}
                >
                    <Container>
                        <FadeIn>
                            <SectionLabel>{servicesLabel}</SectionLabel>
                        </FadeIn>
                        <FadeIn delay={0.1}>
                            <SectionTitle>{servicesTitle}</SectionTitle>
                        </FadeIn>
                        <FadeIn delay={0.2}>
                            <p
                                style={{
                                    fontSize: "1.1rem",
                                    color: textSecondary,
                                    maxWidth: 640,
                                    lineHeight: 1.7,
                                }}
                            >
                                {servicesSubtitle}
                            </p>
                        </FadeIn>
                        <div
                            style={{
                                display: "grid",
                                gridTemplateColumns:
                                    "repeat(auto-fit, minmax(300px, 1fr))",
                                gap: 24,
                                marginTop: 64,
                            }}
                        >
                            {[
                                {
                                    icon: serviceCard1Icon,
                                    title: serviceCard1Title,
                                    body: serviceCard1Body,
                                    img: serviceCard1Image,
                                },
                                {
                                    icon: serviceCard2Icon,
                                    title: serviceCard2Title,
                                    body: serviceCard2Body,
                                    img: serviceCard2Image,
                                },
                                {
                                    icon: serviceCard3Icon,
                                    title: serviceCard3Title,
                                    body: serviceCard3Body,
                                    img: serviceCard3Image,
                                },
                                {
                                    icon: serviceCard4Icon,
                                    title: serviceCard4Title,
                                    body: serviceCard4Body,
                                    img: serviceCard4Image,
                                },
                            ].map((item, i) => (
                                <FadeIn key={i} delay={i * 0.1}>
                                    <Card
                                        style={{
                                            padding: "40px 36px",
                                            background: resolveColor(
                                                serviceCardBg,
                                                bgCard
                                            ),
                                            borderColor: resolveColor(
                                                serviceCardBorder,
                                                borderColor
                                            ),
                                            overflow: "hidden",
                                        }}
                                        onClick={() => navigate("services")}
                                    >
                                        {item.img && item.img.trim() !== "" ? (
                                            <img
                                                src={item.img}
                                                alt=""
                                                style={{
                                                    width: 44,
                                                    height: 44,
                                                    objectFit: "contain",
                                                    marginBottom: 24,
                                                    borderRadius: 8,
                                                }}
                                            />
                                        ) : (
                                            <IconBox size={44}>
                                                {item.icon}
                                            </IconBox>
                                        )}
                                        <h3
                                            style={{
                                                fontFamily: headingFont,
                                                fontSize: `${serviceCardTitleFontSize}rem`,
                                                fontWeight: 600,
                                                marginBottom: 12,
                                            }}
                                        >
                                            {item.title}
                                        </h3>
                                        <p
                                            style={{
                                                fontSize: `${serviceCardBodyFontSize}rem`,
                                                color: textSecondary,
                                                lineHeight: 1.7,
                                                marginBottom: 20,
                                            }}
                                        >
                                            {item.body}
                                        </p>
                                        <span
                                            style={{
                                                fontSize: "0.875rem",
                                                fontWeight: 600,
                                                color: resolveColor(
                                                    serviceLearnMoreColor,
                                                    ac
                                                ),
                                            }}
                                        >
                                            {serviceLearnMoreText}
                                        </span>
                                    </Card>
                                </FadeIn>
                            ))}
                        </div>
                    </Container>
                </section>
            )}

            {/* Speed Callout */}
            {showSpeedCallout && (
                <section
                    style={{
                        padding: `${sectionPadding}px 0`,
                        background: resolveColor(
                            speedBg,
                            `linear-gradient(135deg, ${bgSecondary}, ${bgTertiary})`
                        ),
                        borderTop: `1px solid ${borderColor}`,
                        borderBottom: `1px solid ${borderColor}`,
                        textAlign: "center" as const,
                    }}
                >
                    <Container>
                        <FadeIn>
                            <div
                                style={{
                                    fontFamily: headingFont,
                                    fontSize: speedNumberFontSize,
                                    fontWeight: headingWeight,
                                    color: resolveColor(speedNumberColor, ac),
                                    lineHeight: 1,
                                    marginBottom: 8,
                                }}
                            >
                                {speedNumber}
                            </div>
                            <div
                                style={{
                                    fontFamily: headingFont,
                                    fontSize: speedLabelFontSize,
                                    fontWeight: 600,
                                    marginBottom: 24,
                                    color: resolveColor(
                                        speedLabelColor,
                                        textPrimary
                                    ),
                                }}
                            >
                                {speedLabel}
                            </div>
                            <p
                                style={{
                                    fontSize: "1.1rem",
                                    color: resolveColor(
                                        speedDescColor,
                                        textSecondary
                                    ),
                                    maxWidth: 600,
                                    margin: "0 auto 8px",
                                    lineHeight: 1.7,
                                }}
                            >
                                {speedDesc}
                            </p>
                            <p
                                style={{
                                    fontSize: "0.95rem",
                                    color: resolveColor(
                                        speedSubColor,
                                        textMuted
                                    ),
                                    fontStyle: "italic",
                                }}
                            >
                                {speedSub}
                            </p>
                        </FadeIn>
                    </Container>
                </section>
            )}

            {/* Metrics */}
            {showMetrics && (
                <section
                    style={{
                        padding: "80px 0",
                        background: resolveColor(metricsBg, bgPrimary),
                    }}
                >
                    <Container>
                        <div
                            style={{
                                display: "grid",
                                gridTemplateColumns:
                                    "repeat(auto-fit, minmax(200px, 1fr))",
                                gap: 32,
                            }}
                        >
                            {[
                                {
                                    count: true,
                                    target: metric1Value,
                                    suffix: metric1Suffix,
                                    text: "",
                                    label: metric1Label,
                                    color: resolveColor(metric1Color, ac),
                                },
                                {
                                    count: false,
                                    target: 0,
                                    suffix: "",
                                    text: metric2Text,
                                    label: metric2Label,
                                    color: resolveColor(metric2Color, ac),
                                },
                                {
                                    count: false,
                                    target: 0,
                                    suffix: "",
                                    text: metric3Text,
                                    label: metric3Label,
                                    color: resolveColor(metric3Color, ac),
                                },
                                {
                                    count: true,
                                    target: metric4Value,
                                    suffix: metric4Suffix,
                                    text: "",
                                    label: metric4Label,
                                    color: resolveColor(metric4Color, ac),
                                },
                            ].map((m, i) => (
                                <FadeIn key={i} delay={i * 0.1}>
                                    <div
                                        style={{
                                            textAlign: "center" as const,
                                            padding: 24,
                                        }}
                                    >
                                        <div
                                            style={{
                                                fontFamily: headingFont,
                                                fontSize: metricNumberFontSize,
                                                fontWeight: headingWeight,
                                                color: m.color,
                                                lineHeight: 1,
                                                marginBottom: 8,
                                            }}
                                        >
                                            {m.count ? (
                                                <AnimatedCounter
                                                    target={m.target}
                                                    suffix={m.suffix}
                                                />
                                            ) : (
                                                m.text
                                            )}
                                        </div>
                                        <div
                                            style={{
                                                fontSize: `${metricLabelFontSize}rem`,
                                                color: resolveColor(
                                                    metricLabelColor,
                                                    textSecondary
                                                ),
                                            }}
                                        >
                                            {m.label}
                                        </div>
                                    </div>
                                </FadeIn>
                            ))}
                        </div>
                    </Container>
                </section>
            )}

            {/* Portfolio Preview */}
            {showPortfolioPreview && (
                <section
                    style={{
                        padding: `${sectionPadding}px 0`,
                        background: resolveColor(
                            portfolioPreviewBg,
                            bgSecondary
                        ),
                    }}
                >
                    <Container>
                        <FadeIn>
                            <SectionLabel>{portfolioPreviewLabel}</SectionLabel>
                        </FadeIn>
                        <FadeIn delay={0.1}>
                            <SectionTitle>{portfolioPreviewTitle}</SectionTitle>
                        </FadeIn>
                        <div
                            style={{
                                display: "grid",
                                gridTemplateColumns:
                                    "repeat(auto-fit, minmax(300px, 1fr))",
                                gap: 24,
                                marginTop: 64,
                            }}
                        >
                            {[
                                {
                                    tag: portfolio1Tag,
                                    title: portfolio1Title,
                                    desc: portfolio1Desc,
                                    img: portfolio1Image,
                                    v: 0,
                                },
                                {
                                    tag: portfolio2Tag,
                                    title: portfolio2Title,
                                    desc: portfolio2Desc,
                                    img: portfolio2Image,
                                    v: 1,
                                },
                                {
                                    tag: portfolio3Tag,
                                    title: portfolio3Title,
                                    desc: portfolio3Desc,
                                    img: portfolio3Image,
                                    v: 2,
                                },
                                {
                                    tag: portfolio4Tag,
                                    title: portfolio4Title,
                                    desc: portfolio4Desc,
                                    img: portfolio4Image,
                                    v: 3,
                                },
                            ].map((item, i) => (
                                <FadeIn key={i} delay={i * 0.1}>
                                    <motion.div
                                        whileHover={{ y: -4 }}
                                        onClick={() => navigate("portfolio")}
                                        style={{
                                            borderRadius: cardBorderRadius,
                                            overflow: "hidden",
                                            background: resolveColor(
                                                portfolioCardBg,
                                                bgCard
                                            ),
                                            border: `1px solid ${resolveColor(portfolioCardBorder, borderColor)}`,
                                            cursor: "pointer",
                                        }}
                                    >
                                        <PortfolioImageSlot
                                            src={item.img}
                                            variant={item.v}
                                        />
                                        <div style={{ padding: 24 }}>
                                            <span
                                                style={{
                                                    display: "inline-block",
                                                    padding: "4px 12px",
                                                    background: resolveColor(
                                                        portfolioTagBg,
                                                        acGlow
                                                    ),
                                                    border: `1px solid ${resolveColor(portfolioTagBorder, acBorder)}`,
                                                    borderRadius: 50,
                                                    fontSize: `${portfolioTagFontSize}rem`,
                                                    fontWeight: 600,
                                                    color: resolveColor(
                                                        portfolioTagColor,
                                                        ac
                                                    ),
                                                    marginBottom: 12,
                                                    textTransform:
                                                        "uppercase" as const,
                                                    letterSpacing: 1,
                                                }}
                                            >
                                                {item.tag}
                                            </span>
                                            <h3
                                                style={{
                                                    fontFamily: headingFont,
                                                    fontSize: `${portfolioTitleFontSize}rem`,
                                                    fontWeight: 600,
                                                    marginBottom: 8,
                                                }}
                                            >
                                                {item.title}
                                            </h3>
                                            <p
                                                style={{
                                                    fontSize: `${portfolioDescFontSize}rem`,
                                                    color: textSecondary,
                                                    lineHeight: 1.6,
                                                }}
                                            >
                                                {item.desc}
                                            </p>
                                        </div>
                                    </motion.div>
                                </FadeIn>
                            ))}
                        </div>
                        <FadeIn>
                            <div
                                style={{
                                    textAlign: "center" as const,
                                    marginTop: 48,
                                }}
                            >
                                <BtnSecondary
                                    onClick={() => navigate("portfolio")}
                                >
                                    {portfolioViewAllText}
                                </BtnSecondary>
                            </div>
                        </FadeIn>
                    </Container>
                </section>
            )}

            {/* Process */}
            {showProcess && (
                <section
                    style={{
                        padding: `${sectionPadding}px 0`,
                        background: resolveColor(processBg, bgPrimary),
                    }}
                >
                    <Container>
                        <div style={{ textAlign: "center" as const }}>
                            <FadeIn>
                                <SectionLabel>{processLabel}</SectionLabel>
                            </FadeIn>
                            <FadeIn delay={0.1}>
                                <SectionTitle style={{ textAlign: "center" }}>
                                    {processTitle}
                                </SectionTitle>
                            </FadeIn>
                            <FadeIn delay={0.2}>
                                <p
                                    style={{
                                        fontSize: "1.1rem",
                                        color: textSecondary,
                                        maxWidth: 640,
                                        margin: "0 auto",
                                        lineHeight: 1.7,
                                    }}
                                >
                                    {processSubtitle}
                                </p>
                            </FadeIn>
                        </div>
                        <div
                            style={{
                                display: "grid",
                                gridTemplateColumns:
                                    "repeat(auto-fit, minmax(220px, 1fr))",
                                gap: 32,
                                marginTop: 64,
                            }}
                        >
                            {[
                                {
                                    num: process1Num,
                                    title: process1Title,
                                    body: process1Body,
                                },
                                {
                                    num: process2Num,
                                    title: process2Title,
                                    body: process2Body,
                                },
                                {
                                    num: process3Num,
                                    title: process3Title,
                                    body: process3Body,
                                },
                                {
                                    num: process4Num,
                                    title: process4Title,
                                    body: process4Body,
                                },
                            ].map((step, i) => (
                                <FadeIn key={i} delay={i * 0.1}>
                                    <div
                                        style={{ textAlign: "center" as const }}
                                    >
                                        <div
                                            style={{
                                                width: processStepNumberSize,
                                                height: processStepNumberSize,
                                                background: resolveColor(
                                                    processStepBg,
                                                    bgPrimary
                                                ),
                                                border: `2px solid ${resolveColor(processStepBorder, ac)}`,
                                                borderRadius: "50%",
                                                display: "flex",
                                                alignItems: "center",
                                                justifyContent: "center",
                                                fontFamily: headingFont,
                                                fontSize: "1.25rem",
                                                fontWeight: headingWeight,
                                                color: ac,
                                                margin: "0 auto 24px",
                                            }}
                                        >
                                            {step.num}
                                        </div>
                                        <h3
                                            style={{
                                                fontFamily: headingFont,
                                                fontSize: `${processStepTitleFontSize}rem`,
                                                fontWeight: 600,
                                                marginBottom: 12,
                                            }}
                                        >
                                            {step.title}
                                        </h3>
                                        <p
                                            style={{
                                                fontSize: `${processStepBodyFontSize}rem`,
                                                color: textSecondary,
                                                lineHeight: 1.7,
                                            }}
                                        >
                                            {step.body}
                                        </p>
                                    </div>
                                </FadeIn>
                            ))}
                        </div>
                    </Container>
                </section>
            )}

            {/* Testimonials */}
            {showTestimonials && (
                <section
                    style={{
                        padding: `${sectionPadding}px 0`,
                        background: resolveColor(testimonialsBg, bgSecondary),
                    }}
                >
                    <Container>
                        <FadeIn>
                            <SectionLabel>{testimonialsLabel}</SectionLabel>
                        </FadeIn>
                        <FadeIn delay={0.1}>
                            <SectionTitle>{testimonialsTitle}</SectionTitle>
                        </FadeIn>
                        <div
                            style={{
                                display: "grid",
                                gridTemplateColumns:
                                    "repeat(auto-fit, minmax(300px, 1fr))",
                                gap: 24,
                                marginTop: 64,
                            }}
                        >
                            {[
                                {
                                    text: testimonial1Text,
                                    name: testimonial1Name,
                                    role: testimonial1Role,
                                    initials: testimonial1Initials,
                                    image: testimonial1Image,
                                    stars: testimonial1Stars,
                                },
                                {
                                    text: testimonial2Text,
                                    name: testimonial2Name,
                                    role: testimonial2Role,
                                    initials: testimonial2Initials,
                                    image: testimonial2Image,
                                    stars: testimonial2Stars,
                                },
                                {
                                    text: testimonial3Text,
                                    name: testimonial3Name,
                                    role: testimonial3Role,
                                    initials: testimonial3Initials,
                                    image: testimonial3Image,
                                    stars: testimonial3Stars,
                                },
                            ].map((t, i) => (
                                <FadeIn key={i} delay={i * 0.1}>
                                    <Card
                                        style={{
                                            padding: 36,
                                            background: resolveColor(
                                                testimonialCardBg,
                                                bgCard
                                            ),
                                            borderColor: resolveColor(
                                                testimonialCardBorder,
                                                borderColor
                                            ),
                                        }}
                                    >
                                        <Stars count={t.stars} />
                                        <p
                                            style={{
                                                fontSize: `${testimonialTextFontSize}rem`,
                                                color: resolveColor(
                                                    testimonialTextColor,
                                                    textSecondary
                                                ),
                                                lineHeight: 1.7,
                                                marginBottom: 24,
                                                fontStyle: "italic",
                                            }}
                                        >
                                            {t.text}
                                        </p>
                                        <div
                                            style={{
                                                display: "flex",
                                                alignItems: "center",
                                                gap: 12,
                                            }}
                                        >
                                            <Avatar
                                                image={t.image}
                                                initials={t.initials}
                                            />
                                            <div>
                                                <div
                                                    style={{
                                                        fontWeight: 600,
                                                        fontSize: `${testimonialNameFontSize}rem`,
                                                    }}
                                                >
                                                    {t.name}
                                                </div>
                                                <div
                                                    style={{
                                                        fontSize: `${testimonialRoleFontSize}rem`,
                                                        color: textMuted,
                                                    }}
                                                >
                                                    {t.role}
                                                </div>
                                            </div>
                                        </div>
                                    </Card>
                                </FadeIn>
                            ))}
                        </div>
                    </Container>
                </section>
            )}

            {/* Final CTA */}
            {showFinalCta && (
                <section
                    style={{
                        padding: `${sectionPadding}px 0`,
                        textAlign: "center" as const,
                        background: resolveColor(
                            finalCtaBg,
                            `linear-gradient(135deg, ${bgSecondary}, ${bgPrimary})`
                        ),
                        position: "relative" as const,
                        overflow: "hidden",
                    }}
                >
                    <div
                        style={{
                            position: "absolute",
                            top: "50%",
                            left: "50%",
                            transform: "translate(-50%,-50%)",
                            width: 600,
                            height: 600,
                            background: `radial-gradient(circle, ${resolveColor(finalCtaGlowColor, acGlow)} 0%, transparent 70%)`,
                            pointerEvents: "none" as const,
                        }}
                    />
                    <Container style={{ position: "relative", zIndex: 1 }}>
                        <FadeIn>
                            <h2
                                style={{
                                    fontFamily: headingFont,
                                    fontSize: finalCtaTitleFontSize,
                                    fontWeight: headingWeight,
                                    marginBottom: 16,
                                    color: resolveColor(
                                        finalCtaTitleColor,
                                        textPrimary
                                    ),
                                }}
                            >
                                {finalCtaTitle}
                            </h2>
                            <p
                                style={{
                                    fontSize: "1.1rem",
                                    color: resolveColor(
                                        finalCtaBodyColor,
                                        textSecondary
                                    ),
                                    maxWidth: 580,
                                    margin: "0 auto 40px",
                                    lineHeight: 1.7,
                                }}
                            >
                                {finalCtaBody}
                            </p>
                            <BtnPrimary onClick={() => navigate("contact")}>
                                {finalCtaButton}
                            </BtnPrimary>
                            {showFinalCtaEmail && (
                                <p
                                    style={{
                                        marginTop: 24,
                                        fontSize: "0.9rem",
                                        color: textMuted,
                                    }}
                                >
                                    {finalCtaEmailLabel}{" "}
                                    <a
                                        href={`mailto:${finalCtaEmail}`}
                                        style={{ color: ac }}
                                    >
                                        {finalCtaEmail}
                                    </a>
                                </p>
                            )}
                        </FadeIn>
                    </Container>
                </section>
            )}
        </div>
    )

    // ==========================================
    // PAGE: SERVICES
    // ==========================================
    const renderServices = () => (
        <div style={{ paddingTop: navHeight }}>
            <section
                style={{
                    padding: "100px 0 80px",
                    textAlign: "center" as const,
                }}
            >
                <Container>
                    <FadeIn>
                        <SectionLabel>Services</SectionLabel>
                    </FadeIn>
                    <FadeIn delay={0.1}>
                        <h1
                            style={{
                                fontFamily: headingFont,
                                fontSize: servicesPageTitleFontSize,
                                fontWeight: headingWeight,
                                marginBottom: 16,
                            }}
                        >
                            {servicesPageTitle}
                        </h1>
                    </FadeIn>
                    <FadeIn delay={0.2}>
                        <p
                            style={{
                                fontSize: "1.1rem",
                                color: textSecondary,
                                maxWidth: 680,
                                margin: "0 auto",
                                lineHeight: 1.7,
                            }}
                        >
                            {servicesPageSubtitle}
                        </p>
                    </FadeIn>
                </Container>
            </section>
            <ServiceBlock
                icon={sd1Icon}
                title={sd1Title}
                desc={sd1Desc}
                subServices={[
                    { title: sd1Sub1Title, body: sd1Sub1Body },
                    { title: sd1Sub2Title, body: sd1Sub2Body },
                    { title: sd1Sub3Title, body: sd1Sub3Body },
                    { title: sd1Sub4Title, body: sd1Sub4Body },
                    { title: sd1Sub5Title, body: sd1Sub5Body },
                ]}
            />
            <ServiceBlock
                icon={sd2Icon}
                title={sd2Title}
                desc={sd2Desc}
                bg={bgSecondary}
                subServices={[
                    { title: sd2Sub1Title, body: sd2Sub1Body },
                    { title: sd2Sub2Title, body: sd2Sub2Body },
                    { title: sd2Sub3Title, body: sd2Sub3Body },
                ]}
            />
            <ServiceBlock
                icon={sd3Icon}
                title={sd3Title}
                desc={sd3Desc}
                subServices={[
                    { title: sd3Sub1Title, body: sd3Sub1Body },
                    { title: sd3Sub2Title, body: sd3Sub2Body },
                    { title: sd3Sub3Title, body: sd3Sub3Body },
                ]}
            />
            <ServiceBlock
                icon={sd4Icon}
                title={sd4Title}
                desc={sd4Desc}
                bg={bgSecondary}
                subServices={[
                    { title: sd4Sub1Title, body: sd4Sub1Body },
                    { title: sd4Sub2Title, body: sd4Sub2Body },
                    { title: sd4Sub3Title, body: sd4Sub3Body },
                ]}
            />
            <CtaSection
                title={servicesCtaTitle}
                body={servicesCtaBody}
                button={servicesCtaButton}
                onButtonClick={() => navigate("contact")}
            />
        </div>
    )

    // ==========================================
    // PAGE: PORTFOLIO
    // ==========================================
    const portfolioProjects = [
        {
            id: "1",
            tags: pf1Tags,
            title: pf1Title,
            desc: pf1Desc,
            img: pf1Image,
            v: 0,
        },
        {
            id: "2",
            tags: pf2Tags,
            title: pf2Title,
            desc: pf2Desc,
            img: pf2Image,
            v: 1,
        },
        {
            id: "3",
            tags: pf3Tags,
            title: pf3Title,
            desc: pf3Desc,
            img: pf3Image,
            v: 2,
        },
        {
            id: "4",
            tags: pf4Tags,
            title: pf4Title,
            desc: pf4Desc,
            img: pf4Image,
            v: 3,
        },
        {
            id: "5",
            tags: pf5Tags,
            title: pf5Title,
            desc: pf5Desc,
            img: pf5Image,
            v: 4,
        },
        {
            id: "6",
            tags: pf6Tags,
            title: pf6Title,
            desc: pf6Desc,
            img: pf6Image,
            v: 5,
        },
        {
            id: "7",
            tags: pf7Tags,
            title: pf7Title,
            desc: pf7Desc,
            img: pf7Image,
            v: 6,
        },
        {
            id: "8",
            tags: pf8Tags,
            title: pf8Title,
            desc: pf8Desc,
            img: pf8Image,
            v: 7,
        },
    ]

    const renderPortfolio = () => (
        <div style={{ paddingTop: navHeight }}>
            <section
                style={{
                    padding: "100px 0 80px",
                    textAlign: "center" as const,
                }}
            >
                <Container>
                    <FadeIn>
                        <SectionLabel>Portfolio</SectionLabel>
                    </FadeIn>
                    <FadeIn delay={0.1}>
                        <h1
                            style={{
                                fontFamily: headingFont,
                                fontSize: servicesPageTitleFontSize,
                                fontWeight: headingWeight,
                                marginBottom: 16,
                            }}
                        >
                            {portfolioPageTitle}
                        </h1>
                    </FadeIn>
                    <FadeIn delay={0.2}>
                        <p style={{ fontSize: "1.1rem", color: textSecondary }}>
                            {portfolioPageSubtitle}
                        </p>
                    </FadeIn>
                </Container>
            </section>
            <section style={{ paddingBottom: 80 }}>
                <Container>
                    <FadeIn>
                        <div
                            style={{
                                display: "flex",
                                justifyContent: "center",
                                gap: 12,
                                marginBottom: 48,
                                flexWrap: "wrap" as const,
                            }}
                        >
                            {[
                                { key: "all", label: portfolioFilterAll },
                                {
                                    key: "healthcare",
                                    label: portfolioFilterHealthcare,
                                },
                                {
                                    key: "real-estate",
                                    label: portfolioFilterRealEstate,
                                },
                                { key: "ai", label: portfolioFilterAI },
                                {
                                    key: "content",
                                    label: portfolioFilterContent,
                                },
                            ].map((f) => (
                                <button
                                    key={f.key}
                                    onClick={() => setActiveFilter(f.key)}
                                    style={{
                                        padding: "8px 20px",
                                        border: `1px solid ${activeFilter === f.key ? resolveColor(filterBtnActiveBg, ac) : resolveColor(filterBtnBorder, borderColor)}`,
                                        borderRadius: buttonBorderRadius,
                                        fontSize: "0.825rem",
                                        fontWeight: 500,
                                        color:
                                            activeFilter === f.key
                                                ? filterBtnActiveColor
                                                : resolveColor(
                                                      filterBtnColor,
                                                      textSecondary
                                                  ),
                                        background:
                                            activeFilter === f.key
                                                ? resolveColor(
                                                      filterBtnActiveBg,
                                                      ac
                                                  )
                                                : filterBtnBg,
                                        cursor: "pointer",
                                        fontFamily: bodyFont,
                                    }}
                                >
                                    {f.label}
                                </button>
                            ))}
                        </div>
                    </FadeIn>
                    <div
                        style={{
                            display: "grid",
                            gridTemplateColumns:
                                "repeat(auto-fit, minmax(340px, 1fr))",
                            gap: 32,
                        }}
                    >
                        {portfolioProjects
                            .filter(
                                (p) =>
                                    activeFilter === "all" ||
                                    p.tags.includes(activeFilter)
                            )
                            .map((project, i) => (
                                <FadeIn key={project.id} delay={i * 0.05}>
                                    <motion.div
                                        whileHover={{ y: -4 }}
                                        onClick={() =>
                                            setActiveCaseStudy(
                                                caseStudies[project.id]
                                            )
                                        }
                                        style={{
                                            background: bgCard,
                                            border: `1px solid ${borderColor}`,
                                            borderRadius: cardBorderRadius,
                                            overflow: "hidden",
                                            cursor: "pointer",
                                        }}
                                    >
                                        <PortfolioImageSlot
                                            src={project.img}
                                            variant={project.v}
                                            height={280}
                                        />
                                        <div style={{ padding: 28 }}>
                                            <div
                                                style={{
                                                    display: "flex",
                                                    gap: 8,
                                                    marginBottom: 12,
                                                    flexWrap: "wrap" as const,
                                                }}
                                            >
                                                {project.tags
                                                    .split(",")
                                                    .map((tag) => (
                                                        <span
                                                            key={tag}
                                                            style={{
                                                                padding:
                                                                    "4px 12px",
                                                                background:
                                                                    acGlow,
                                                                border: `1px solid ${acBorder}`,
                                                                borderRadius: 50,
                                                                fontSize:
                                                                    "0.7rem",
                                                                fontWeight: 600,
                                                                color: ac,
                                                                textTransform:
                                                                    "uppercase" as const,
                                                                letterSpacing: 1,
                                                            }}
                                                        >
                                                            {tag
                                                                .trim()
                                                                .replace(
                                                                    "-",
                                                                    " "
                                                                )}
                                                        </span>
                                                    ))}
                                            </div>
                                            <h3
                                                style={{
                                                    fontFamily: headingFont,
                                                    fontSize: "1.25rem",
                                                    fontWeight: 600,
                                                    marginBottom: 8,
                                                }}
                                            >
                                                {project.title}
                                            </h3>
                                            <p
                                                style={{
                                                    fontSize: "0.9rem",
                                                    color: textSecondary,
                                                    lineHeight: 1.6,
                                                    marginBottom: 16,
                                                }}
                                            >
                                                {project.desc}
                                            </p>
                                            <span
                                                style={{
                                                    fontSize: "0.875rem",
                                                    fontWeight: 600,
                                                    color: ac,
                                                }}
                                            >
                                                View Case Study →
                                            </span>
                                        </div>
                                    </motion.div>
                                </FadeIn>
                            ))}
                    </div>
                </Container>
            </section>
            <CtaSection
                title={portfolioCtaTitle}
                body={portfolioCtaBody}
                button={portfolioCtaButton}
                onButtonClick={() => navigate("contact")}
            />
        </div>
    )

    // ==========================================
    // PAGE: ABOUT
    // ==========================================
    const renderAbout = () => (
        <div style={{ paddingTop: navHeight }}>
            <section style={{ padding: "100px 0 80px" }}>
                <Container>
                    <FadeIn>
                        <SectionLabel>About</SectionLabel>
                    </FadeIn>
                    <FadeIn delay={0.1}>
                        <h1
                            style={{
                                fontFamily: headingFont,
                                fontSize: aboutPageTitleFontSize,
                                fontWeight: headingWeight,
                                marginBottom: 8,
                            }}
                        >
                            {aboutPageTitle}
                        </h1>
                    </FadeIn>
                    <FadeIn delay={0.2}>
                        <p style={{ fontSize: "1.1rem", color: textSecondary }}>
                            {aboutPageSubtitle}
                        </p>
                    </FadeIn>
                </Container>
            </section>
            <section>
                <Container>
                    <div
                        style={{
                            display: "grid",
                            gridTemplateColumns: "1fr 1.5fr",
                            gap: 64,
                            paddingBottom: 80,
                        }}
                        className="about-grid"
                    >
                        <FadeIn direction="left">
                            <div
                                style={{
                                    width: "100%",
                                    aspectRatio: aboutPhotoAspectRatio,
                                    borderRadius: aboutPhotoBorderRadius,
                                    overflow: "hidden",
                                    border: `1px solid ${borderColor}`,
                                    background: bgTertiary,
                                }}
                            >
                                <ImageOrPlaceholder
                                    src={aboutPhoto}
                                    alt="David"
                                    placeholderIcon={aboutPhotoPlaceholderIcon}
                                    placeholderText={aboutPhotoPlaceholderText}
                                    accentColor={ac}
                                    bgColor={bgTertiary}
                                />
                            </div>
                        </FadeIn>
                        <FadeIn direction="right">
                            <div>
                                {[
                                    aboutBio1,
                                    aboutBio2,
                                    aboutBio3,
                                    aboutBio4,
                                    aboutBio5,
                                ].map((p, i) => (
                                    <p
                                        key={i}
                                        style={{
                                            fontSize: `${i === 0 ? aboutBio1FontSize : aboutBioFontSize}rem`,
                                            color:
                                                i === 0
                                                    ? textPrimary
                                                    : textSecondary,
                                            lineHeight: 1.8,
                                            marginBottom: 20,
                                        }}
                                    >
                                        {p}
                                    </p>
                                ))}
                            </div>
                        </FadeIn>
                    </div>
                </Container>
            </section>
            {/* Principles */}
            <section
                style={{
                    padding: "80px 0",
                    borderTop: `1px solid ${borderColor}`,
                }}
            >
                <Container>
                    <FadeIn>
                        <SectionLabel>{principlesLabel}</SectionLabel>
                    </FadeIn>
                    <FadeIn delay={0.1}>
                        <SectionTitle>{principlesTitle}</SectionTitle>
                    </FadeIn>
                    <div
                        style={{
                            display: "grid",
                            gridTemplateColumns:
                                "repeat(auto-fit, minmax(300px, 1fr))",
                            gap: 32,
                            marginTop: 48,
                        }}
                    >
                        {[
                            {
                                num: principle1Num,
                                title: principle1Title,
                                body: principle1Body,
                            },
                            {
                                num: principle2Num,
                                title: principle2Title,
                                body: principle2Body,
                            },
                            {
                                num: principle3Num,
                                title: principle3Title,
                                body: principle3Body,
                            },
                        ].map((p, i) => (
                            <FadeIn key={i} delay={i * 0.1}>
                                <Card
                                    style={{
                                        padding: 36,
                                        background: resolveColor(
                                            principleCardBg,
                                            bgCard
                                        ),
                                    }}
                                >
                                    <div
                                        style={{
                                            fontFamily: headingFont,
                                            fontSize: `${principleNumFontSize}rem`,
                                            fontWeight: headingWeight,
                                            color: resolveColor(
                                                principleNumColor,
                                                ac
                                            ),
                                            marginBottom: 16,
                                            opacity: 0.5,
                                        }}
                                    >
                                        {p.num}
                                    </div>
                                    <h3
                                        style={{
                                            fontFamily: headingFont,
                                            fontSize: `${principleTitleFontSize}rem`,
                                            fontWeight: 600,
                                            marginBottom: 12,
                                        }}
                                    >
                                        {p.title}
                                    </h3>
                                    <p
                                        style={{
                                            fontSize: `${principleBodyFontSize}rem`,
                                            color: textSecondary,
                                            lineHeight: 1.7,
                                        }}
                                    >
                                        {p.body}
                                    </p>
                                </Card>
                            </FadeIn>
                        ))}
                    </div>
                </Container>
            </section>
            {/* Focus */}
            <section
                style={{
                    padding: "80px 0",
                    borderTop: `1px solid ${borderColor}`,
                }}
            >
                <Container>
                    <FadeIn>
                        <SectionLabel>{focusLabel}</SectionLabel>
                    </FadeIn>
                    <FadeIn delay={0.1}>
                        <SectionTitle>{focusTitle}</SectionTitle>
                    </FadeIn>
                    <div style={{ marginTop: 48, display: "grid", gap: 24 }}>
                        {[focus1, focus2, focus3].map((f, i) => (
                            <FadeIn key={i} delay={i * 0.1}>
                                <Card
                                    style={{
                                        display: "flex",
                                        alignItems: "flex-start",
                                        gap: 20,
                                        padding: 32,
                                        background: resolveColor(
                                            focusItemBg,
                                            bgCard
                                        ),
                                    }}
                                >
                                    <span
                                        style={{
                                            fontFamily: headingFont,
                                            fontSize: "1.5rem",
                                            fontWeight: headingWeight,
                                            color: resolveColor(
                                                focusNumColor,
                                                ac
                                            ),
                                            minWidth: 36,
                                        }}
                                    >
                                        {i + 1}
                                    </span>
                                    <p
                                        style={{
                                            fontSize: "0.95rem",
                                            color: textSecondary,
                                            lineHeight: 1.7,
                                        }}
                                    >
                                        {f}
                                    </p>
                                </Card>
                            </FadeIn>
                        ))}
                    </div>
                </Container>
            </section>
            <CtaSection
                title={aboutCtaTitle}
                body={aboutCtaBody}
                button={aboutCtaButton}
                onButtonClick={() => navigate("contact")}
            />
        </div>
    )

    // ==========================================
    // PAGE: CONTACT
    // ==========================================
    const renderContact = () => (
        <div style={{ paddingTop: navHeight }}>
            <section
                style={{
                    padding: "100px 0 80px",
                    textAlign: "center" as const,
                }}
            >
                <Container>
                    <FadeIn>
                        <SectionLabel>Contact</SectionLabel>
                    </FadeIn>
                    <FadeIn delay={0.1}>
                        <h1
                            style={{
                                fontFamily: headingFont,
                                fontSize: contactPageTitleFontSize,
                                fontWeight: headingWeight,
                                marginBottom: 16,
                            }}
                        >
                            {contactPageTitle}
                        </h1>
                    </FadeIn>
                    <FadeIn delay={0.2}>
                        <p
                            style={{
                                fontSize: "1.1rem",
                                color: textSecondary,
                                maxWidth: 560,
                                margin: "0 auto",
                                lineHeight: 1.7,
                            }}
                        >
                            {contactPageSubtitle}
                        </p>
                    </FadeIn>
                </Container>
            </section>
            <section>
                <Container>
                    <div
                        style={{
                            display: "grid",
                            gridTemplateColumns:
                                "repeat(auto-fit, minmax(340px, 1fr))",
                            gap: 48,
                            paddingBottom: 80,
                        }}
                    >
                        <FadeIn direction="left">
                            <div
                                style={{
                                    background: resolveColor(
                                        bookingCardBg,
                                        bgCard
                                    ),
                                    border: `1px solid ${borderColor}`,
                                    borderRadius: cardBorderRadius,
                                    padding: 40,
                                }}
                            >
                                <h3
                                    style={{
                                        fontFamily: headingFont,
                                        fontSize: "1.25rem",
                                        fontWeight: 600,
                                        marginBottom: 8,
                                    }}
                                >
                                    {bookingTitle}
                                </h3>
                                <p
                                    style={{
                                        fontSize: "0.9rem",
                                        color: textSecondary,
                                        marginBottom: 32,
                                        lineHeight: 1.6,
                                    }}
                                >
                                    {bookingDesc}
                                </p>
                                <div
                                    style={{
                                        background: bgTertiary,
                                        border: `1px solid ${borderColor}`,
                                        borderRadius: 8,
                                        padding: 40,
                                        textAlign: "center" as const,
                                    }}
                                >
                                    <div
                                        style={{
                                            fontSize: "3rem",
                                            marginBottom: 16,
                                            opacity: 0.4,
                                        }}
                                    >
                                        {bookingPlaceholderIcon}
                                    </div>
                                    <p
                                        style={{
                                            fontSize: "0.9rem",
                                            color: textMuted,
                                            marginBottom: 16,
                                        }}
                                    >
                                        {bookingPlaceholderText}
                                    </p>
                                    <a
                                        href={calendlyUrl}
                                        target="_blank"
                                        rel="noopener noreferrer"
                                        style={{
                                            display: "inline-flex",
                                            alignItems: "center",
                                            gap: 8,
                                            padding: "14px 32px",
                                            background: ac,
                                            color: "#FFF",
                                            fontWeight: 600,
                                            fontSize: "0.95rem",
                                            borderRadius: buttonBorderRadius,
                                            textDecoration: "none",
                                            border: `2px solid ${ac}`,
                                        }}
                                    >
                                        {bookingLinkText}
                                    </a>
                                </div>
                            </div>
                        </FadeIn>
                        <FadeIn direction="right">
                            <div
                                style={{
                                    background: resolveColor(
                                        formCardBg,
                                        bgCard
                                    ),
                                    border: `1px solid ${borderColor}`,
                                    borderRadius: cardBorderRadius,
                                    padding: 40,
                                }}
                            >
                                {!formSubmitted ? (
                                    <>
                                        <h3
                                            style={{
                                                fontFamily: headingFont,
                                                fontSize: "1.25rem",
                                                fontWeight: 600,
                                                marginBottom: 32,
                                            }}
                                        >
                                            {formTitle}
                                        </h3>
                                        <form
                                            onSubmit={(e) => {
                                                e.preventDefault()
                                                setFormSubmitted(true)
                                            }}
                                        >
                                            {[
                                                {
                                                    label: formNameLabel,
                                                    type: "text",
                                                    required: true,
                                                    placeholder:
                                                        formNamePlaceholder,
                                                },
                                                {
                                                    label: formEmailLabel,
                                                    type: "email",
                                                    required: true,
                                                    placeholder:
                                                        formEmailPlaceholder,
                                                },
                                                {
                                                    label: formPhoneLabel,
                                                    type: "tel",
                                                    required: false,
                                                    placeholder:
                                                        formPhonePlaceholder,
                                                },
                                            ].map((field, i) => (
                                                <div
                                                    key={i}
                                                    style={{ marginBottom: 20 }}
                                                >
                                                    <label
                                                        style={{
                                                            display: "block",
                                                            fontSize:
                                                                "0.825rem",
                                                            fontWeight: 600,
                                                            marginBottom: 8,
                                                            color: textSecondary,
                                                        }}
                                                    >
                                                        {field.label}
                                                    </label>
                                                    <input
                                                        type={field.type}
                                                        required={
                                                            field.required
                                                        }
                                                        placeholder={
                                                            field.placeholder
                                                        }
                                                        style={{
                                                            width: "100%",
                                                            padding:
                                                                "12px 16px",
                                                            background:
                                                                resolveColor(
                                                                    formInputBg,
                                                                    bgPrimary
                                                                ),
                                                            border: `1px solid ${resolveColor(formInputBorder, borderColor)}`,
                                                            borderRadius: 8,
                                                            color: resolveColor(
                                                                formInputColor,
                                                                textPrimary
                                                            ),
                                                            fontFamily:
                                                                bodyFont,
                                                            fontSize: "0.9rem",
                                                            outline: "none",
                                                            boxSizing:
                                                                "border-box" as const,
                                                        }}
                                                    />
                                                </div>
                                            ))}
                                            <div style={{ marginBottom: 20 }}>
                                                <label
                                                    style={{
                                                        display: "block",
                                                        fontSize: "0.825rem",
                                                        fontWeight: 600,
                                                        marginBottom: 8,
                                                        color: textSecondary,
                                                    }}
                                                >
                                                    {formBusinessLabel}
                                                </label>
                                                <select
                                                    style={{
                                                        width: "100%",
                                                        padding: "12px 16px",
                                                        background:
                                                            resolveColor(
                                                                formInputBg,
                                                                bgPrimary
                                                            ),
                                                        border: `1px solid ${resolveColor(formInputBorder, borderColor)}`,
                                                        borderRadius: 8,
                                                        color: textSecondary,
                                                        fontFamily: bodyFont,
                                                        fontSize: "0.9rem",
                                                        outline: "none",
                                                        boxSizing:
                                                            "border-box" as const,
                                                    }}
                                                >
                                                    <option value="">
                                                        {formBusinessDefault}
                                                    </option>
                                                    <option value="healthcare">
                                                        {formBusinessOption1}
                                                    </option>
                                                    <option value="real-estate">
                                                        {formBusinessOption2}
                                                    </option>
                                                    <option value="other">
                                                        {formBusinessOption3}
                                                    </option>
                                                </select>
                                            </div>
                                            <div style={{ marginBottom: 20 }}>
                                                <label
                                                    style={{
                                                        display: "block",
                                                        fontSize: "0.825rem",
                                                        fontWeight: 600,
                                                        marginBottom: 8,
                                                        color: textSecondary,
                                                    }}
                                                >
                                                    {formMessageLabel}
                                                </label>
                                                <textarea
                                                    required
                                                    placeholder={
                                                        formMessagePlaceholder
                                                    }
                                                    style={{
                                                        width: "100%",
                                                        padding: "12px 16px",
                                                        background:
                                                            resolveColor(
                                                                formInputBg,
                                                                bgPrimary
                                                            ),
                                                        border: `1px solid ${resolveColor(formInputBorder, borderColor)}`,
                                                        borderRadius: 8,
                                                        color: resolveColor(
                                                            formInputColor,
                                                            textPrimary
                                                        ),
                                                        fontFamily: bodyFont,
                                                        fontSize: "0.9rem",
                                                        outline: "none",
                                                        minHeight: 100,
                                                        resize: "vertical" as const,
                                                        boxSizing:
                                                            "border-box" as const,
                                                    }}
                                                />
                                            </div>
                                            <motion.button
                                                whileHover={{
                                                    y: -2,
                                                    boxShadow: `0 8px 30px ${ac}4D`,
                                                }}
                                                type="submit"
                                                style={{
                                                    width: "100%",
                                                    padding: "14px 32px",
                                                    background: resolveColor(
                                                        formSubmitBg,
                                                        ac
                                                    ),
                                                    color: formSubmitColor,
                                                    fontWeight: 600,
                                                    fontSize: "0.95rem",
                                                    borderRadius:
                                                        buttonBorderRadius,
                                                    border: "none",
                                                    cursor: "pointer",
                                                    fontFamily: bodyFont,
                                                }}
                                            >
                                                {formSubmitText}
                                            </motion.button>
                                        </form>
                                    </>
                                ) : (
                                    <div
                                        style={{
                                            textAlign: "center" as const,
                                            padding: 40,
                                        }}
                                    >
                                        <div
                                            style={{
                                                fontSize: "3rem",
                                                marginBottom: 16,
                                            }}
                                        >
                                            {formSuccessIcon}
                                        </div>
                                        <h4
                                            style={{
                                                fontFamily: headingFont,
                                                fontSize: "1.25rem",
                                                fontWeight: 600,
                                                marginBottom: 8,
                                            }}
                                        >
                                            {formSuccessTitle}
                                        </h4>
                                        <p
                                            style={{
                                                fontSize: "0.9rem",
                                                color: textSecondary,
                                            }}
                                        >
                                            {formSuccessBody}
                                        </p>
                                    </div>
                                )}
                            </div>
                        </FadeIn>
                    </div>
                    <div
                        style={{
                            display: "grid",
                            gridTemplateColumns:
                                "repeat(auto-fit, minmax(250px, 1fr))",
                            gap: 24,
                            paddingBottom: 80,
                        }}
                    >
                        {[
                            {
                                icon: contactEmailIcon,
                                label: contactEmailLabel,
                                value: contactEmail,
                                isLink: true,
                            },
                            {
                                icon: contactLocationIcon,
                                label: contactLocationLabel,
                                value: contactLocation,
                                isLink: false,
                            },
                            {
                                icon: contactResponseIcon,
                                label: contactResponseLabel,
                                value: contactResponse,
                                isLink: false,
                            },
                        ].map((item, i) => (
                            <FadeIn key={i} delay={i * 0.1}>
                                <Card
                                    style={{
                                        padding: 28,
                                        textAlign: "center" as const,
                                        background: resolveColor(
                                            contactCardBg,
                                            bgCard
                                        ),
                                    }}
                                >
                                    <div
                                        style={{
                                            fontSize: "1.5rem",
                                            marginBottom: 12,
                                        }}
                                    >
                                        {item.icon}
                                    </div>
                                    <h4
                                        style={{
                                            fontSize: "0.8rem",
                                            fontWeight: 600,
                                            letterSpacing: 2,
                                            textTransform: "uppercase" as const,
                                            color: textMuted,
                                            marginBottom: 8,
                                        }}
                                    >
                                        {item.label}
                                    </h4>
                                    <p
                                        style={{
                                            fontSize: "0.95rem",
                                            color: textSecondary,
                                        }}
                                    >
                                        {item.isLink ? (
                                            <a
                                                href={`mailto:${item.value}`}
                                                style={{ color: ac }}
                                            >
                                                {item.value}
                                            </a>
                                        ) : (
                                            item.value
                                        )}
                                    </p>
                                </Card>
                            </FadeIn>
                        ))}
                    </div>
                </Container>
            </section>
        </div>
    )

    // ==========================================
    // PAGE: BLOG
    // ==========================================
    const blogItems = [
        {
            id: "1",
            icon: blog1Icon,
            date: blog1Date,
            title: blog1Title,
            excerpt: blog1Excerpt,
            img: blog1Image,
        },
        {
            id: "2",
            icon: blog2Icon,
            date: blog2Date,
            title: blog2Title,
            excerpt: blog2Excerpt,
            img: blog2Image,
        },
        {
            id: "3",
            icon: blog3Icon,
            date: blog3Date,
            title: blog3Title,
            excerpt: blog3Excerpt,
            img: blog3Image,
        },
        {
            id: "4",
            icon: blog4Icon,
            date: blog4Date,
            title: blog4Title,
            excerpt: blog4Excerpt,
            img: blog4Image,
        },
        {
            id: "5",
            icon: blog5Icon,
            date: blog5Date,
            title: blog5Title,
            excerpt: blog5Excerpt,
            img: blog5Image,
        },
    ]

    const renderBlog = () => (
        <div style={{ paddingTop: navHeight }}>
            <section
                style={{
                    padding: "100px 0 80px",
                    textAlign: "center" as const,
                }}
            >
                <Container>
                    <FadeIn>
                        <SectionLabel>Insights</SectionLabel>
                    </FadeIn>
                    <FadeIn delay={0.1}>
                        <h1
                            style={{
                                fontFamily: headingFont,
                                fontSize: blogPageTitleFontSize,
                                fontWeight: headingWeight,
                                marginBottom: 16,
                            }}
                        >
                            {blogPageTitle}
                        </h1>
                    </FadeIn>
                    <FadeIn delay={0.2}>
                        <p
                            style={{
                                fontSize: "1.1rem",
                                color: textSecondary,
                                maxWidth: 560,
                                margin: "0 auto",
                            }}
                        >
                            {blogPageSubtitle}
                        </p>
                    </FadeIn>
                </Container>
            </section>
            <section>
                <Container>
                    <div
                        style={{
                            display: "grid",
                            gridTemplateColumns:
                                "repeat(auto-fit, minmax(340px, 1fr))",
                            gap: 32,
                            paddingBottom: 80,
                        }}
                    >
                        {blogItems.map((blog, i) => (
                            <FadeIn key={blog.id} delay={i * 0.05}>
                                <motion.div
                                    whileHover={{ y: -4 }}
                                    onClick={() =>
                                        setActiveBlog(blogArticles[blog.id])
                                    }
                                    style={{
                                        background: resolveColor(
                                            blogCardBg,
                                            bgCard
                                        ),
                                        border: `1px solid ${resolveColor(blogCardBorder, borderColor)}`,
                                        borderRadius: cardBorderRadius,
                                        overflow: "hidden",
                                        cursor: "pointer",
                                    }}
                                >
                                    <div
                                        style={{
                                            height: 200,
                                            background: resolveColor(
                                                blogImageBg,
                                                bgTertiary
                                            ),
                                            display: "flex",
                                            alignItems: "center",
                                            justifyContent: "center",
                                        }}
                                    >
                                        {blog.img && blog.img.trim() !== "" ? (
                                            <img
                                                src={blog.img}
                                                alt=""
                                                style={{
                                                    width: "100%",
                                                    height: "100%",
                                                    objectFit: "cover",
                                                }}
                                            />
                                        ) : (
                                            <span
                                                style={{
                                                    fontSize: "2.5rem",
                                                    opacity: 0.2,
                                                }}
                                            >
                                                {blog.icon}
                                            </span>
                                        )}
                                    </div>
                                    <div style={{ padding: 28 }}>
                                        <div
                                            style={{
                                                fontSize: "0.75rem",
                                                color: resolveColor(
                                                    blogDateColor,
                                                    textMuted
                                                ),
                                                marginBottom: 12,
                                            }}
                                        >
                                            {blog.date}
                                        </div>
                                        <h3
                                            style={{
                                                fontFamily: headingFont,
                                                fontSize: `${blogTitleFontSize}rem`,
                                                fontWeight: 600,
                                                marginBottom: 8,
                                                lineHeight: 1.4,
                                            }}
                                        >
                                            {blog.title}
                                        </h3>
                                        <p
                                            style={{
                                                fontSize: `${blogExcerptFontSize}rem`,
                                                color: textSecondary,
                                                lineHeight: 1.6,
                                                marginBottom: 16,
                                            }}
                                        >
                                            {blog.excerpt}
                                        </p>
                                        <span
                                            style={{
                                                fontSize: "0.85rem",
                                                fontWeight: 600,
                                                color: resolveColor(
                                                    blogReadMoreColor,
                                                    ac
                                                ),
                                            }}
                                        >
                                            {blogReadMoreText}
                                        </span>
                                    </div>
                                </motion.div>
                            </FadeIn>
                        ))}
                    </div>
                </Container>
            </section>
        </div>
    )

    // ==========================================
    // MODAL: CASE STUDY
    // ==========================================
    const renderCaseStudyModal = () => (
        <AnimatePresence>
            {activeCaseStudy && (
                <motion.div
                    initial={{ opacity: 0 }}
                    animate={{ opacity: 1 }}
                    exit={{ opacity: 0 }}
                    onClick={() => setActiveCaseStudy(null)}
                    style={{
                        position: "fixed" as const,
                        inset: 0,
                        background: csModalBg,
                        backdropFilter: "blur(10px)",
                        zIndex: 2000,
                        overflowY: "auto" as const,
                        padding: "40px 24px",
                    }}
                >
                    <motion.div
                        initial={{ opacity: 0, y: 30 }}
                        animate={{ opacity: 1, y: 0 }}
                        exit={{ opacity: 0, y: 30 }}
                        onClick={(e) => e.stopPropagation()}
                        style={{
                            maxWidth: 800,
                            margin: "0 auto",
                            background: resolveColor(csContentBg, bgSecondary),
                            border: `1px solid ${borderColor}`,
                            borderRadius: cardBorderRadius,
                            padding: 48,
                            position: "relative" as const,
                        }}
                    >
                        <button
                            onClick={() => setActiveCaseStudy(null)}
                            style={{
                                position: "absolute" as const,
                                top: 20,
                                right: 20,
                                width: 40,
                                height: 40,
                                display: "flex",
                                alignItems: "center",
                                justifyContent: "center",
                                background: bgTertiary,
                                border: `1px solid ${borderColor}`,
                                borderRadius: "50%",
                                color: textSecondary,
                                fontSize: "1.2rem",
                                cursor: "pointer",
                            }}
                        >
                            {csCloseIcon}
                        </button>
                        <SectionLabel>{activeCaseStudy.tag}</SectionLabel>
                        <SectionTitle style={{ marginTop: 8 }}>
                            {activeCaseStudy.title}
                        </SectionTitle>
                        <div
                            style={{
                                display: "grid",
                                gridTemplateColumns: "1fr 1fr",
                                gap: 16,
                                margin: "32px 0",
                                padding: 24,
                                background: bgCard,
                                borderRadius: 8,
                            }}
                        >
                            <div>
                                <label
                                    style={{
                                        fontSize: "0.7rem",
                                        fontWeight: 600,
                                        letterSpacing: 2,
                                        textTransform: "uppercase" as const,
                                        color: textMuted,
                                        display: "block",
                                        marginBottom: 4,
                                    }}
                                >
                                    {csIndustryLabel}
                                </label>
                                <span
                                    style={{
                                        fontSize: "0.9rem",
                                        color: textSecondary,
                                    }}
                                >
                                    {activeCaseStudy.industry}
                                </span>
                            </div>
                            <div>
                                <label
                                    style={{
                                        fontSize: "0.7rem",
                                        fontWeight: 600,
                                        letterSpacing: 2,
                                        textTransform: "uppercase" as const,
                                        color: textMuted,
                                        display: "block",
                                        marginBottom: 4,
                                    }}
                                >
                                    {csScopeLabel}
                                </label>
                                <span
                                    style={{
                                        fontSize: "0.9rem",
                                        color: textSecondary,
                                    }}
                                >
                                    {activeCaseStudy.scope}
                                </span>
                            </div>
                        </div>
                        {[
                            {
                                title: csChallengeLabel,
                                body: activeCaseStudy.challenge,
                            },
                            {
                                title: csSolutionLabel,
                                body: activeCaseStudy.solution,
                            },
                            {
                                title: csOutcomeLabel,
                                body: activeCaseStudy.outcome,
                            },
                        ].map((s, i) => (
                            <div key={i} style={{ marginBottom: 32 }}>
                                <h3
                                    style={{
                                        fontFamily: headingFont,
                                        fontSize: "1.15rem",
                                        fontWeight: 600,
                                        marginBottom: 12,
                                        color: ac,
                                    }}
                                >
                                    {s.title}
                                </h3>
                                <p
                                    style={{
                                        fontSize: "0.95rem",
                                        color: textSecondary,
                                        lineHeight: 1.8,
                                    }}
                                >
                                    {s.body}
                                </p>
                            </div>
                        ))}
                    </motion.div>
                </motion.div>
            )}
        </AnimatePresence>
    )

    // ==========================================
    // MODAL: BLOG
    // ==========================================
    const renderBlogModal = () => (
        <AnimatePresence>
            {activeBlog && (
                <motion.div
                    initial={{ opacity: 0 }}
                    animate={{ opacity: 1 }}
                    exit={{ opacity: 0 }}
                    onClick={() => setActiveBlog(null)}
                    style={{
                        position: "fixed" as const,
                        inset: 0,
                        background: blogModalBg,
                        backdropFilter: "blur(10px)",
                        zIndex: 2000,
                        overflowY: "auto" as const,
                        padding: "40px 24px",
                    }}
                >
                    <motion.div
                        initial={{ opacity: 0, y: 30 }}
                        animate={{ opacity: 1, y: 0 }}
                        exit={{ opacity: 0, y: 30 }}
                        onClick={(e) => e.stopPropagation()}
                        style={{
                            maxWidth: 720,
                            margin: "0 auto",
                            background: resolveColor(
                                blogModalContentBg,
                                bgSecondary
                            ),
                            border: `1px solid ${borderColor}`,
                            borderRadius: cardBorderRadius,
                            padding: 48,
                            position: "relative" as const,
                        }}
                    >
                        <button
                            onClick={() => setActiveBlog(null)}
                            style={{
                                position: "absolute" as const,
                                top: 20,
                                right: 20,
                                width: 40,
                                height: 40,
                                display: "flex",
                                alignItems: "center",
                                justifyContent: "center",
                                background: bgTertiary,
                                border: `1px solid ${borderColor}`,
                                borderRadius: "50%",
                                color: textSecondary,
                                fontSize: "1.2rem",
                                cursor: "pointer",
                            }}
                        >
                            ✕
                        </button>
                        <h2
                            style={{
                                fontFamily: headingFont,
                                fontSize: `${blogModalTitleFontSize}rem`,
                                fontWeight: headingWeight,
                                marginBottom: 24,
                                lineHeight: 1.3,
                                paddingRight: 40,
                            }}
                        >
                            {activeBlog.title}
                        </h2>
                        <div
                            style={{
                                fontSize: "0.8rem",
                                color: textMuted,
                                marginBottom: 32,
                            }}
                        >
                            {activeBlog.date}
                        </div>
                        {activeBlog.body.split("\n\n").map((para, i) => (
                            <p
                                key={i}
                                style={{
                                    fontSize: "1rem",
                                    color: textSecondary,
                                    lineHeight: 1.9,
                                    marginBottom: 20,
                                }}
                            >
                                {para}
                            </p>
                        ))}
                    </motion.div>
                </motion.div>
            )}
        </AnimatePresence>
    )

    // ==========================================
    // NAV
    // ==========================================
    const renderNav = () => (
        <nav
            style={{
                position: "fixed" as const,
                top: 0,
                left: 0,
                right: 0,
                height: navHeight,
                background: navBg,
                backdropFilter: "blur(20px)",
                WebkitBackdropFilter: "blur(20px)",
                borderBottom: `1px solid ${navBorderColor}`,
                zIndex: 1000,
            }}
        >
            <div
                style={{
                    maxWidth: containerMaxWidth,
                    margin: "0 auto",
                    padding: "0 24px",
                    height: "100%",
                    display: "flex",
                    alignItems: "center",
                    justifyContent: "space-between",
                }}
            >
                <div
                    onClick={() => navigate("home")}
                    style={{
                        cursor: "pointer",
                        display: "flex",
                        alignItems: "center",
                        gap: 4,
                    }}
                >
                    {logoImage && logoImage.trim() !== "" ? (
                        <img
                            src={logoImage}
                            alt={navLogoText}
                            style={{
                                height: navLogoFontSize * 24,
                                objectFit: "contain",
                            }}
                        />
                    ) : (
                        <span
                            style={{
                                fontFamily: headingFont,
                                fontSize: `${navLogoFontSize}rem`,
                                fontWeight: navLogoFontWeight,
                                color: navLogoColor,
                                letterSpacing: -0.5,
                            }}
                        >
                            {navLogoText}
                            <span
                                style={{
                                    color: resolveColor(navLogoDotColor, ac),
                                }}
                            >
                                {navLogoDot}
                            </span>
                        </span>
                    )}
                </div>
                <div
                    style={{ display: "flex", alignItems: "center", gap: 36 }}
                    className="nav-links-desktop"
                >
                    {navItems.map((item) => (
                        <button
                            key={item.id}
                            onClick={() => navigate(item.id)}
                            style={{
                                fontSize: `${navLinkFontSize}rem`,
                                fontWeight: 500,
                                color:
                                    currentPage === item.id
                                        ? resolveColor(
                                              navLinkActiveColor,
                                              textPrimary
                                          )
                                        : resolveColor(
                                              navLinkColor,
                                              textSecondary
                                          ),
                                background: "none",
                                border: "none",
                                cursor: "pointer",
                                fontFamily: bodyFont,
                                padding: 0,
                            }}
                        >
                            {item.label}
                        </button>
                    ))}
                    {showNavCta && (
                        <button
                            onClick={() => navigate("contact")}
                            style={{
                                padding: "10px 24px",
                                background: resolveColor(navCtaBg, ac),
                                color: navCtaColor,
                                borderRadius: buttonBorderRadius,
                                fontWeight: 600,
                                fontSize: `${navCtaFontSize}rem`,
                                cursor: "pointer",
                                border: "none",
                                fontFamily: bodyFont,
                            }}
                        >
                            {navCtaText}
                        </button>
                    )}
                </div>
                <button
                    onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
                    style={{
                        display: "none",
                        flexDirection: "column" as const,
                        gap: 5,
                        cursor: "pointer",
                        padding: 4,
                        background: "none",
                        border: "none",
                    }}
                    className="hamburger-btn"
                >
                    <span
                        style={{
                            width: 24,
                            height: 2,
                            background: textPrimary,
                            borderRadius: 2,
                            transition: "all 0.2s",
                            transform: mobileMenuOpen
                                ? "rotate(45deg) translate(5px,5px)"
                                : "none",
                        }}
                    />
                    <span
                        style={{
                            width: 24,
                            height: 2,
                            background: textPrimary,
                            borderRadius: 2,
                            transition: "all 0.2s",
                            opacity: mobileMenuOpen ? 0 : 1,
                        }}
                    />
                    <span
                        style={{
                            width: 24,
                            height: 2,
                            background: textPrimary,
                            borderRadius: 2,
                            transition: "all 0.2s",
                            transform: mobileMenuOpen
                                ? "rotate(-45deg) translate(5px,-5px)"
                                : "none",
                        }}
                    />
                </button>
            </div>
            <AnimatePresence>
                {mobileMenuOpen && (
                    <motion.div
                        initial={{ opacity: 0 }}
                        animate={{ opacity: 1 }}
                        exit={{ opacity: 0 }}
                        style={{
                            position: "fixed" as const,
                            inset: 0,
                            background: "rgba(13,13,13,0.98)",
                            display: "flex",
                            flexDirection: "column" as const,
                            alignItems: "center",
                            justifyContent: "center",
                            gap: 32,
                            zIndex: 999,
                        }}
                    >
                        {navItems.map((item) => (
                            <button
                                key={item.id}
                                onClick={() => navigate(item.id)}
                                style={{
                                    fontFamily: headingFont,
                                    fontSize: "1.75rem",
                                    fontWeight: 600,
                                    color:
                                        currentPage === item.id
                                            ? textPrimary
                                            : textSecondary,
                                    background: "none",
                                    border: "none",
                                    cursor: "pointer",
                                }}
                            >
                                {item.label}
                            </button>
                        ))}
                        <BtnPrimary
                            onClick={() => navigate("contact")}
                            style={{ marginTop: 16 }}
                        >
                            {navCtaText}
                        </BtnPrimary>
                    </motion.div>
                )}
            </AnimatePresence>
        </nav>
    )

    // ==========================================
    // FOOTER
    // ==========================================
    const renderFooter = () => (
        <footer
            style={{
                padding: "40px 0",
                borderTop: `1px solid ${resolveColor(footerBorderColor, borderColor)}`,
                background: resolveColor(footerBg, bgPrimary),
            }}
        >
            <Container>
                <div
                    style={{
                        display: "flex",
                        alignItems: "center",
                        justifyContent: "space-between",
                        flexWrap: "wrap" as const,
                        gap: 24,
                    }}
                >
                    <div
                        style={{
                            fontSize: "0.8rem",
                            color: resolveColor(footerCopyColor, textMuted),
                        }}
                    >
                        {footerCopy}
                    </div>
                    {showFooterLinks && (
                        <div style={{ display: "flex", gap: 24 }}>
                            {navItems.map((item) => (
                                <button
                                    key={item.id}
                                    onClick={() => navigate(item.id)}
                                    style={{
                                        fontSize: "0.8rem",
                                        color: resolveColor(
                                            footerLinkColor,
                                            textMuted
                                        ),
                                        background: "none",
                                        border: "none",
                                        cursor: "pointer",
                                        fontFamily: bodyFont,
                                        padding: 0,
                                    }}
                                >
                                    {item.label}
                                </button>
                            ))}
                        </div>
                    )}
                    {showFooterSocial && (
                        <div style={{ display: "flex", gap: 12 }}>
                            {[
                                {
                                    label: socialLinkedinLabel,
                                    url: socialLinkedinUrl,
                                },
                                {
                                    label: socialYoutubeLabel,
                                    url: socialYoutubeUrl,
                                },
                                {
                                    label: socialTwitterLabel,
                                    url: socialTwitterUrl,
                                },
                                ...(socialGithubUrl
                                    ? [
                                          {
                                              label: socialGithubLabel,
                                              url: socialGithubUrl,
                                          },
                                      ]
                                    : []),
                            ].map((s) => (
                                <a
                                    key={s.label}
                                    href={s.url}
                                    target="_blank"
                                    rel="noopener noreferrer"
                                    style={{
                                        width: socialIconSize,
                                        height: socialIconSize,
                                        display: "flex",
                                        alignItems: "center",
                                        justifyContent: "center",
                                        border: `1px solid ${resolveColor(socialIconBorder, borderColor)}`,
                                        borderRadius: "50%",
                                        color: resolveColor(
                                            socialIconColor,
                                            textMuted
                                        ),
                                        fontSize: "0.85rem",
                                        textDecoration: "none",
                                        background: socialIconBg,
                                    }}
                                >
                                    {s.label}
                                </a>
                            ))}
                        </div>
                    )}
                </div>
            </Container>
        </footer>
    )

    // ==========================================
    // PAGE ROUTER
    // ==========================================
    const pages: Record<string, () => JSX.Element> = {
        home: renderHome,
        services: renderServices,
        portfolio: renderPortfolio,
        about: renderAbout,
        contact: renderContact,
        blog: renderBlog,
    }

    // ==========================================
    // MAIN RENDER
    // ==========================================
    return (
        <div
            style={{
                fontFamily: bodyFont,
                background: bgPrimary,
                color: textPrimary,
                lineHeight: 1.6,
                overflowX: "hidden" as const,
                width: "100%",
                minHeight: "100vh",
                fontSize: baseFontSize,
            }}
        >
            <style>{`
                @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&family=Space+Grotesk:wght@400;500;600;700&display=swap');
                * { box-sizing: border-box; margin: 0; padding: 0; }
                a { text-decoration: none; color: inherit; }
                input:focus, select:focus, textarea:focus { border-color: ${ac} !important; }
                ::selection { background: ${ac}; color: white; }
                @media (max-width: 768px) {
                    .nav-links-desktop { display: none !important; }
                    .hamburger-btn { display: flex !important; }
                    .about-grid { grid-template-columns: 1fr !important; }
                }
                @media (min-width: 769px) {
                    .hamburger-btn { display: none !important; }
                }
            `}</style>
            {renderNav()}
            <AnimatePresence mode="wait">
                <motion.div
                    key={currentPage}
                    initial={{ opacity: 0, y: 10 }}
                    animate={{ opacity: 1, y: 0 }}
                    exit={{ opacity: 0, y: -10 }}
                    transition={{ duration: 0.3 }}
                >
                    {(pages[currentPage] || renderHome)()}
                </motion.div>
            </AnimatePresence>
            {renderFooter()}
            {renderCaseStudyModal()}
            {renderBlogModal()}
        </div>
    )
}

// ============================================
// FRAMER PROPERTY CONTROLS — EVERYTHING EDITABLE
// ============================================
addPropertyControls(AIAAWebsite, {
    // ─── GLOBAL COLORS ───
    accentColor: {
        type: ControlType.Color,
        title: "🎨 Accent Color",
        defaultValue: "#0066FF",
    },
    accentHoverColor: {
        type: ControlType.Color,
        title: "🎨 Accent Hover",
        defaultValue: "#0052CC",
    },
    bgPrimary: {
        type: ControlType.Color,
        title: "🎨 BG Primary",
        defaultValue: "#0D0D0D",
    },
    bgSecondary: {
        type: ControlType.Color,
        title: "🎨 BG Secondary",
        defaultValue: "#111111",
    },
    bgTertiary: {
        type: ControlType.Color,
        title: "🎨 BG Tertiary",
        defaultValue: "#1A1A1A",
    },
    bgCard: {
        type: ControlType.Color,
        title: "🎨 BG Card",
        defaultValue: "#161616",
    },
    bgCardHover: {
        type: ControlType.Color,
        title: "🎨 BG Card Hover",
        defaultValue: "#1E1E1E",
    },
    textPrimary: {
        type: ControlType.Color,
        title: "🎨 Text Primary",
        defaultValue: "#FFFFFF",
    },
    textSecondary: {
        type: ControlType.Color,
        title: "🎨 Text Secondary",
        defaultValue: "#B0B0B0",
    },
    textMuted: {
        type: ControlType.Color,
        title: "🎨 Text Muted",
        defaultValue: "#666666",
    },
    borderColor: {
        type: ControlType.Color,
        title: "🎨 Border Color",
        defaultValue: "#222222",
    },
    borderLightColor: {
        type: ControlType.Color,
        title: "🎨 Border Light",
        defaultValue: "#333333",
    },
    starColor: {
        type: ControlType.Color,
        title: "🎨 Star Color",
        defaultValue: "#FFB800",
    },
    gradientSecondary: {
        type: ControlType.Color,
        title: "🎨 Gradient 2nd",
        defaultValue: "#00AAFF",
    },

    // ─── GLOBAL TYPOGRAPHY ───
    headingFont: {
        type: ControlType.String,
        title: "🔤 Heading Font",
        defaultValue: "'Space Grotesk', 'Inter', -apple-system, sans-serif",
    },
    bodyFont: {
        type: ControlType.String,
        title: "🔤 Body Font",
        defaultValue: "'Inter', 'SF Pro Display', -apple-system, sans-serif",
    },
    headingWeight: {
        type: ControlType.Number,
        title: "🔤 Heading Weight",
        defaultValue: 700,
        min: 100,
        max: 900,
        step: 100,
    },
    baseFontSize: {
        type: ControlType.Number,
        title: "🔤 Base Font Size",
        defaultValue: 16,
        min: 12,
        max: 24,
    },

    // ─── GLOBAL SPACING ───
    sectionPadding: {
        type: ControlType.Number,
        title: "📐 Section Padding",
        defaultValue: 120,
        min: 40,
        max: 200,
    },
    containerMaxWidth: {
        type: ControlType.Number,
        title: "📐 Max Width",
        defaultValue: 1200,
        min: 800,
        max: 1600,
    },
    cardBorderRadius: {
        type: ControlType.Number,
        title: "📐 Card Radius",
        defaultValue: 12,
        min: 0,
        max: 32,
    },
    buttonBorderRadius: {
        type: ControlType.Number,
        title: "📐 Button Radius",
        defaultValue: 50,
        min: 0,
        max: 50,
    },
    navHeight: {
        type: ControlType.Number,
        title: "📐 Nav Height",
        defaultValue: 72,
        min: 48,
        max: 100,
    },

    // ─── GLOBAL IMAGES ───
    logoImage: { type: ControlType.Image, title: "🖼️ Logo Image" },
    ogImage: { type: ControlType.Image, title: "🖼️ OG Image" },

    // ─── NAV ───
    navBg: {
        type: ControlType.String,
        title: "🧭 Nav BG",
        defaultValue: "rgba(13,13,13,0.9)",
    },
    navLogoText: {
        type: ControlType.String,
        title: "🧭 Logo Text",
        defaultValue: "AIAA",
    },
    navLogoDot: {
        type: ControlType.String,
        title: "🧭 Logo Dot",
        defaultValue: ".",
    },
    navLogoColor: {
        type: ControlType.Color,
        title: "🧭 Logo Color",
        defaultValue: "#FFFFFF",
    },
    navLogoDotColor: {
        type: ControlType.Color,
        title: "🧭 Logo Dot Color",
        defaultValue: "#0066FF",
    },
    navLogoFontSize: {
        type: ControlType.Number,
        title: "🧭 Logo Size",
        defaultValue: 1.5,
        min: 0.8,
        max: 3,
        step: 0.1,
    },
    navLinkHome: {
        type: ControlType.String,
        title: "🧭 Link: Home",
        defaultValue: "Home",
    },
    navLinkServices: {
        type: ControlType.String,
        title: "🧭 Link: Services",
        defaultValue: "Services",
    },
    navLinkPortfolio: {
        type: ControlType.String,
        title: "🧭 Link: Portfolio",
        defaultValue: "Portfolio",
    },
    navLinkAbout: {
        type: ControlType.String,
        title: "🧭 Link: About",
        defaultValue: "About",
    },
    navLinkBlog: {
        type: ControlType.String,
        title: "🧭 Link: Blog",
        defaultValue: "Insights",
    },
    navLinkContact: {
        type: ControlType.String,
        title: "🧭 Link: Contact",
        defaultValue: "Contact",
    },
    navCtaText: {
        type: ControlType.String,
        title: "🧭 CTA Text",
        defaultValue: "Book a Call",
    },
    navLinkFontSize: {
        type: ControlType.Number,
        title: "🧭 Link Size",
        defaultValue: 0.875,
        min: 0.6,
        max: 1.5,
        step: 0.025,
    },
    showNavCta: {
        type: ControlType.Boolean,
        title: "🧭 Show CTA",
        defaultValue: true,
    },

    // ─── HERO ───
    heroTag: {
        type: ControlType.String,
        title: "🦸 Tag Text",
        defaultValue: "AI Automation Agency — Los Angeles",
    },
    heroHeadline1: {
        type: ControlType.String,
        title: "🦸 Headline 1",
        defaultValue: "I Build the AI Systems That ",
    },
    heroHighlight: {
        type: ControlType.String,
        title: "🦸 Highlight",
        defaultValue: "Run Your Business",
    },
    heroHeadline2: {
        type: ControlType.String,
        title: "🦸 Headline 2",
        defaultValue: " While You Sleep.",
    },
    heroSubheadline: {
        type: ControlType.String,
        title: "🦸 Subheadline",
        defaultValue:
            "AI-powered automation, high-converting web platforms, and scalable digital infrastructure.",
    },
    heroCta1Text: {
        type: ControlType.String,
        title: "🦸 CTA 1 Text",
        defaultValue: "See My Work",
    },
    heroCta2Text: {
        type: ControlType.String,
        title: "🦸 CTA 2 Text",
        defaultValue: "Book a Strategy Call",
    },
    showParticles: {
        type: ControlType.Boolean,
        title: "🦸 Particles",
        defaultValue: true,
    },
    heroBgImage: { type: ControlType.Image, title: "🦸 BG Image" },
    heroContentAlign: {
        type: ControlType.Enum,
        title: "🦸 Align",
        options: ["left", "center"],
        defaultValue: "left",
    },
    heroHeadlineFontSize: {
        type: ControlType.String,
        title: "🦸 H1 Size",
        defaultValue: "clamp(2.5rem, 5.5vw, 4.5rem)",
    },
    heroSubFontSize: {
        type: ControlType.Number,
        title: "🦸 Sub Size",
        defaultValue: 1.2,
        min: 0.8,
        max: 2,
        step: 0.05,
    },
    heroGradientOpacity: {
        type: ControlType.Number,
        title: "🦸 Glow Opacity",
        defaultValue: 0.15,
        min: 0,
        max: 1,
        step: 0.05,
    },
    showHeroCta1Icon: {
        type: ControlType.Boolean,
        title: "🦸 CTA1 Icon",
        defaultValue: true,
    },

    // ─── TRUST STRIP ───
    showTrustStrip: {
        type: ControlType.Boolean,
        title: "🤝 Show Trust",
        defaultValue: true,
    },
    trustLabel: {
        type: ControlType.String,
        title: "🤝 Label",
        defaultValue: "Systems Built For",
    },
    trust1: {
        type: ControlType.String,
        title: "🤝 Name 1",
        defaultValue: "Classic Clinic",
    },
    trust2: {
        type: ControlType.String,
        title: "🤝 Name 2",
        defaultValue: "Dental Clinic of Compton",
    },
    trust3: {
        type: ControlType.String,
        title: "🤝 Name 3",
        defaultValue: "Dream Realty LA",
    },
    trust4: {
        type: ControlType.String,
        title: "🤝 Name 4",
        defaultValue: "Toma Barseghian",
    },
    trust5: {
        type: ControlType.String,
        title: "🤝 Name 5",
        defaultValue: "Frank Lopez",
    },
    trust6: {
        type: ControlType.String,
        title: "🤝 Name 6",
        defaultValue: "Alina Muradyan",
    },
    trustLogo1: { type: ControlType.Image, title: "🤝 Logo 1" },
    trustLogo2: { type: ControlType.Image, title: "🤝 Logo 2" },
    trustLogo3: { type: ControlType.Image, title: "🤝 Logo 3" },
    trustLogo4: { type: ControlType.Image, title: "🤝 Logo 4" },
    trustLogo5: { type: ControlType.Image, title: "🤝 Logo 5" },
    trustLogo6: { type: ControlType.Image, title: "🤝 Logo 6" },
    trustLogoHeight: {
        type: ControlType.Number,
        title: "🤝 Logo Height",
        defaultValue: 24,
        min: 12,
        max: 64,
    },

    // ─── PROBLEMS ───
    showProblems: {
        type: ControlType.Boolean,
        title: "⚠️ Show Problems",
        defaultValue: true,
    },
    problemsLabel: {
        type: ControlType.String,
        title: "⚠️ Label",
        defaultValue: "The Problem",
    },
    problemsTitle: {
        type: ControlType.String,
        title: "⚠️ Title",
        defaultValue: "Your Business Is Bleeding Revenue.",
    },
    problemsSubtitle: {
        type: ControlType.String,
        title: "⚠️ Subtitle",
        defaultValue: "Most businesses are losing thousands to inefficiency.",
    },
    problem1Icon: {
        type: ControlType.String,
        title: "⚠️ P1 Icon",
        defaultValue: "⏱️",
    },
    problem1Title: {
        type: ControlType.String,
        title: "⚠️ P1 Title",
        defaultValue: "You're Losing Hours to Manual Tasks",
    },
    problem1Body: {
        type: ControlType.String,
        title: "⚠️ P1 Body",
        defaultValue:
            "Every minute spent on manual follow-ups is revenue walking out the door.",
    },
    problem2Icon: {
        type: ControlType.String,
        title: "⚠️ P2 Icon",
        defaultValue: "📉",
    },
    problem2Title: {
        type: ControlType.String,
        title: "⚠️ P2 Title",
        defaultValue: "You're Dropping Leads",
    },
    problem2Body: {
        type: ControlType.String,
        title: "⚠️ P2 Body",
        defaultValue: "Slow response times are silently killing your growth.",
    },
    problem3Icon: {
        type: ControlType.String,
        title: "⚠️ P3 Icon",
        defaultValue: "⭐",
    },
    problem3Title: {
        type: ControlType.String,
        title: "⚠️ P3 Title",
        defaultValue: "Your Online Presence Isn't Working",
    },
    problem3Body: {
        type: ControlType.String,
        title: "⚠️ P3 Body",
        defaultValue:
            "An outdated website costs you trust before a prospect calls.",
    },

    // ─── SERVICES OVERVIEW ───
    showServicesOverview: {
        type: ControlType.Boolean,
        title: "🛠️ Show Services",
        defaultValue: true,
    },
    servicesLabel: {
        type: ControlType.String,
        title: "🛠️ Label",
        defaultValue: "What I Build",
    },
    servicesTitle: {
        type: ControlType.String,
        title: "🛠️ Title",
        defaultValue: "End-to-End Systems That Scale",
    },
    serviceCard1Icon: {
        type: ControlType.String,
        title: "🛠️ S1 Icon",
        defaultValue: "🤖",
    },
    serviceCard1Title: {
        type: ControlType.String,
        title: "🛠️ S1 Title",
        defaultValue: "AI Automation",
    },
    serviceCard1Body: {
        type: ControlType.String,
        title: "🛠️ S1 Body",
        defaultValue: "Intelligent bots and automated scheduling.",
    },
    serviceCard1Image: { type: ControlType.Image, title: "🛠️ S1 Image" },
    serviceCard2Icon: {
        type: ControlType.String,
        title: "🛠️ S2 Icon",
        defaultValue: "🌐",
    },
    serviceCard2Title: {
        type: ControlType.String,
        title: "🛠️ S2 Title",
        defaultValue: "Web Design",
    },
    serviceCard2Body: {
        type: ControlType.String,
        title: "🛠️ S2 Body",
        defaultValue: "High-converting websites.",
    },
    serviceCard2Image: { type: ControlType.Image, title: "🛠️ S2 Image" },
    serviceCard3Icon: {
        type: ControlType.String,
        title: "🛠️ S3 Icon",
        defaultValue: "🎬",
    },
    serviceCard3Title: {
        type: ControlType.String,
        title: "🛠️ S3 Title",
        defaultValue: "Content Strategy",
    },
    serviceCard3Body: {
        type: ControlType.String,
        title: "🛠️ S3 Body",
        defaultValue: "Scalable AI content pipelines.",
    },
    serviceCard3Image: { type: ControlType.Image, title: "🛠️ S3 Image" },
    serviceCard4Icon: {
        type: ControlType.String,
        title: "🛠️ S4 Icon",
        defaultValue: "📊",
    },
    serviceCard4Title: {
        type: ControlType.String,
        title: "🛠️ S4 Title",
        defaultValue: "Consulting",
    },
    serviceCard4Body: {
        type: ControlType.String,
        title: "🛠️ S4 Body",
        defaultValue: "Niche selection and pricing models.",
    },
    serviceCard4Image: { type: ControlType.Image, title: "🛠️ S4 Image" },
    serviceLearnMoreText: {
        type: ControlType.String,
        title: "🛠️ Learn More",
        defaultValue: "Learn More →",
    },

    // ─── SPEED ───
    showSpeedCallout: {
        type: ControlType.Boolean,
        title: "⚡ Show Speed",
        defaultValue: true,
    },
    speedNumber: {
        type: ControlType.String,
        title: "⚡ Number",
        defaultValue: "30 Minutes.",
    },
    speedLabel: {
        type: ControlType.String,
        title: "⚡ Label",
        defaultValue: "From Zero to Live Deployment.",
    },
    speedDesc: {
        type: ControlType.String,
        title: "⚡ Desc",
        defaultValue:
            "That's how long it took me to build and launch a chatbot.",
    },
    speedSub: {
        type: ControlType.String,
        title: "⚡ Sub",
        defaultValue: "Speed without compromise is the standard.",
    },

    // ─── METRICS ───
    showMetrics: {
        type: ControlType.Boolean,
        title: "📊 Show Metrics",
        defaultValue: true,
    },
    metric1Value: {
        type: ControlType.Number,
        title: "📊 M1 Value",
        defaultValue: 8,
    },
    metric1Suffix: {
        type: ControlType.String,
        title: "📊 M1 Suffix",
        defaultValue: "+",
    },
    metric1Label: {
        type: ControlType.String,
        title: "📊 M1 Label",
        defaultValue: "Web Platforms Delivered",
    },
    metric2Text: {
        type: ControlType.String,
        title: "📊 M2 Text",
        defaultValue: "24/7",
    },
    metric2Label: {
        type: ControlType.String,
        title: "📊 M2 Label",
        defaultValue: "AI Systems Running Live",
    },
    metric3Text: {
        type: ControlType.String,
        title: "📊 M3 Text",
        defaultValue: "0",
    },
    metric3Label: {
        type: ControlType.String,
        title: "📊 M3 Label",
        defaultValue: "Dropped Leads",
    },
    metric4Value: {
        type: ControlType.Number,
        title: "📊 M4 Value",
        defaultValue: 3,
    },
    metric4Label: {
        type: ControlType.String,
        title: "📊 M4 Label",
        defaultValue: "Videos Published Daily",
    },

    // ─── PORTFOLIO PREVIEW ───
    showPortfolioPreview: {
        type: ControlType.Boolean,
        title: "🖼️ Show Preview",
        defaultValue: true,
    },
    portfolioPreviewLabel: {
        type: ControlType.String,
        title: "🖼️ Label",
        defaultValue: "Selected Work",
    },
    portfolioPreviewTitle: {
        type: ControlType.String,
        title: "🖼️ Title",
        defaultValue: "Projects That Speak for Themselves",
    },
    portfolio1Title: {
        type: ControlType.String,
        title: "🖼️ P1 Title",
        defaultValue: "Classic Clinic",
    },
    portfolio1Tag: {
        type: ControlType.String,
        title: "🖼️ P1 Tag",
        defaultValue: "Healthcare",
    },
    portfolio1Desc: {
        type: ControlType.String,
        title: "🖼️ P1 Desc",
        defaultValue: "Full healthcare web design.",
    },
    portfolio1Image: { type: ControlType.Image, title: "🖼️ P1 Image" },
    portfolio2Title: {
        type: ControlType.String,
        title: "🖼️ P2 Title",
        defaultValue: "Dental Clinic",
    },
    portfolio2Tag: {
        type: ControlType.String,
        title: "🖼️ P2 Tag",
        defaultValue: "Healthcare",
    },
    portfolio2Desc: {
        type: ControlType.String,
        title: "🖼️ P2 Desc",
        defaultValue: "Dental practice website.",
    },
    portfolio2Image: { type: ControlType.Image, title: "🖼️ P2 Image" },
    portfolio3Title: {
        type: ControlType.String,
        title: "🖼️ P3 Title",
        defaultValue: "Dream Realty LA",
    },
    portfolio3Tag: {
        type: ControlType.String,
        title: "🖼️ P3 Tag",
        defaultValue: "Real Estate",
    },
    portfolio3Desc: {
        type: ControlType.String,
        title: "🖼️ P3 Desc",
        defaultValue: "Luxury real estate platform.",
    },
    portfolio3Image: { type: ControlType.Image, title: "🖼️ P3 Image" },
    portfolio4Title: {
        type: ControlType.String,
        title: "🖼️ P4 Title",
        defaultValue: "30-Min Chatbot",
    },
    portfolio4Tag: {
        type: ControlType.String,
        title: "🖼️ P4 Tag",
        defaultValue: "AI Automation",
    },
    portfolio4Desc: {
        type: ControlType.String,
        title: "🖼️ P4 Desc",
        defaultValue: "AI chatbot in 30 minutes.",
    },
    portfolio4Image: { type: ControlType.Image, title: "🖼️ P4 Image" },
    portfolioViewAllText: {
        type: ControlType.String,
        title: "🖼️ View All",
        defaultValue: "View All Projects →",
    },

    // ─── PROCESS ───
    showProcess: {
        type: ControlType.Boolean,
        title: "⚙️ Show Process",
        defaultValue: true,
    },
    processLabel: {
        type: ControlType.String,
        title: "⚙️ Label",
        defaultValue: "The Process",
    },
    processTitle: {
        type: ControlType.String,
        title: "⚙️ Title",
        defaultValue: "From Strategy to Deployment",
    },
    process1Title: {
        type: ControlType.String,
        title: "⚙️ S1 Title",
        defaultValue: "Discovery & Audit",
    },
    process1Body: {
        type: ControlType.String,
        title: "⚙️ S1 Body",
        defaultValue: "Analyze operations and pain points.",
    },
    process2Title: {
        type: ControlType.String,
        title: "⚙️ S2 Title",
        defaultValue: "Architecture & Strategy",
    },
    process2Body: {
        type: ControlType.String,
        title: "⚙️ S2 Body",
        defaultValue: "Map out the complete system.",
    },
    process3Title: {
        type: ControlType.String,
        title: "⚙️ S3 Title",
        defaultValue: "Rapid Build & Deploy",
    },
    process3Body: {
        type: ControlType.String,
        title: "⚙️ S3 Body",
        defaultValue: "Execute fast without compromise.",
    },
    process4Title: {
        type: ControlType.String,
        title: "⚙️ S4 Title",
        defaultValue: "Optimization & Scale",
    },
    process4Body: {
        type: ControlType.String,
        title: "⚙️ S4 Body",
        defaultValue: "Monitor and expand capabilities.",
    },

    // ─── TESTIMONIALS ───
    showTestimonials: {
        type: ControlType.Boolean,
        title: "💬 Show Testimonials",
        defaultValue: true,
    },
    testimonialsLabel: {
        type: ControlType.String,
        title: "💬 Label",
        defaultValue: "What Clients Say",
    },
    testimonialsTitle: {
        type: ControlType.String,
        title: "💬 Title",
        defaultValue: "Trusted by Industry Leaders",
    },
    testimonial1Text: {
        type: ControlType.String,
        title: "💬 T1 Text",
        defaultValue: '"David completely transformed our operations."',
    },
    testimonial1Name: {
        type: ControlType.String,
        title: "💬 T1 Name",
        defaultValue: "Dr. Maria Rodriguez",
    },
    testimonial1Role: {
        type: ControlType.String,
        title: "💬 T1 Role",
        defaultValue: "Director, Classic Clinic",
    },
    testimonial1Initials: {
        type: ControlType.String,
        title: "💬 T1 Initials",
        defaultValue: "MR",
    },
    testimonial1Image: { type: ControlType.Image, title: "💬 T1 Photo" },
    testimonial1Stars: {
        type: ControlType.Number,
        title: "💬 T1 Stars",
        defaultValue: 5,
        min: 0,
        max: 5,
    },
    testimonial2Text: {
        type: ControlType.String,
        title: "💬 T2 Text",
        defaultValue: '"Our website generates more leads in a week."',
    },
    testimonial2Name: {
        type: ControlType.String,
        title: "💬 T2 Name",
        defaultValue: "Toma Barseghian",
    },
    testimonial2Role: {
        type: ControlType.String,
        title: "💬 T2 Role",
        defaultValue: "Real Estate Agent",
    },
    testimonial2Initials: {
        type: ControlType.String,
        title: "💬 T2 Initials",
        defaultValue: "TB",
    },
    testimonial2Image: { type: ControlType.Image, title: "💬 T2 Photo" },
    testimonial2Stars: {
        type: ControlType.Number,
        title: "💬 T2 Stars",
        defaultValue: 5,
        min: 0,
        max: 5,
    },
    testimonial3Text: {
        type: ControlType.String,
        title: "💬 T3 Text",
        defaultValue: '"The speed was unbelievable."',
    },
    testimonial3Name: {
        type: ControlType.String,
        title: "💬 T3 Name",
        defaultValue: "Dr. James Chen",
    },
    testimonial3Role: {
        type: ControlType.String,
        title: "💬 T3 Role",
        defaultValue: "Owner, Dental Clinic",
    },
    testimonial3Initials: {
        type: ControlType.String,
        title: "💬 T3 Initials",
        defaultValue: "JC",
    },
    testimonial3Image: { type: ControlType.Image, title: "💬 T3 Photo" },
    testimonial3Stars: {
        type: ControlType.Number,
        title: "💬 T3 Stars",
        defaultValue: 5,
        min: 0,
        max: 5,
    },

    // ─── FINAL CTA ───
    showFinalCta: {
        type: ControlType.Boolean,
        title: "🎯 Show CTA",
        defaultValue: true,
    },
    finalCtaTitle: {
        type: ControlType.String,
        title: "🎯 Title",
        defaultValue: "Ready to Stop Losing Revenue?",
    },
    finalCtaBody: {
        type: ControlType.String,
        title: "🎯 Body",
        defaultValue: "Let's build the system that runs your business 24/7.",
    },
    finalCtaButton: {
        type: ControlType.String,
        title: "🎯 Button",
        defaultValue: "Book Your Strategy Call",
    },
    finalCtaEmail: {
        type: ControlType.String,
        title: "🎯 Email",
        defaultValue: "david@aiaa.agency",
    },
    showFinalCtaEmail: {
        type: ControlType.Boolean,
        title: "🎯 Show Email",
        defaultValue: true,
    },

    // ─── ABOUT ───
    aboutPageTitle: {
        type: ControlType.String,
        title: "👤 Title",
        defaultValue: "About David",
    },
    aboutPageSubtitle: {
        type: ControlType.String,
        title: "👤 Subtitle",
        defaultValue: "Engineer. Strategist. Builder.",
    },
    aboutPhoto: { type: ControlType.Image, title: "👤 Photo" },
    aboutPhotoPlaceholderIcon: {
        type: ControlType.String,
        title: "👤 Photo Icon",
        defaultValue: "👤",
    },
    aboutBio1: {
        type: ControlType.String,
        title: "👤 Bio 1",
        defaultValue: "I'm David — an AI systems engineer.",
    },
    aboutBio2: {
        type: ControlType.String,
        title: "👤 Bio 2",
        defaultValue: "I started with a simple observation...",
    },
    aboutBio3: {
        type: ControlType.String,
        title: "👤 Bio 3",
        defaultValue: "So I build the solutions.",
    },
    aboutBio4: {
        type: ControlType.String,
        title: "👤 Bio 4",
        defaultValue: "I design AI-powered systems...",
    },
    aboutBio5: {
        type: ControlType.String,
        title: "👤 Bio 5",
        defaultValue: "I'm not a freelancer. I'm an engineer.",
    },
    principle1Title: {
        type: ControlType.String,
        title: "👤 Princ 1",
        defaultValue: '"Speed Is a Feature."',
    },
    principle1Body: {
        type: ControlType.String,
        title: "👤 Princ 1 Body",
        defaultValue: "Full chatbot in 30 minutes.",
    },
    principle2Title: {
        type: ControlType.String,
        title: "👤 Princ 2",
        defaultValue: '"Every System Must Pay for Itself."',
    },
    principle2Body: {
        type: ControlType.String,
        title: "👤 Princ 2 Body",
        defaultValue: "Clear revenue impact.",
    },
    principle3Title: {
        type: ControlType.String,
        title: "👤 Princ 3",
        defaultValue: '"Build Once. Run Forever."',
    },
    principle3Body: {
        type: ControlType.String,
        title: "👤 Princ 3 Body",
        defaultValue: "Permanent infrastructure.",
    },
    focus1: {
        type: ControlType.String,
        title: "👤 Focus 1",
        defaultValue: "Scaling AIAA.",
    },
    focus2: {
        type: ControlType.String,
        title: "👤 Focus 2",
        defaultValue: "Deepening AI content pipeline.",
    },
    focus3: {
        type: ControlType.String,
        title: "👤 Focus 3",
        defaultValue: "Pushing the technical frontier.",
    },

    // ─── CONTACT ───
    contactPageTitle: {
        type: ControlType.String,
        title: "📞 Title",
        defaultValue: "Let's Build Something",
    },
    contactPageSubtitle: {
        type: ControlType.String,
        title: "📞 Subtitle",
        defaultValue: "It starts with a conversation.",
    },
    bookingTitle: {
        type: ControlType.String,
        title: "📞 Booking Title",
        defaultValue: "Book a Free Strategy Call",
    },
    bookingDesc: {
        type: ControlType.String,
        title: "📞 Booking Desc",
        defaultValue: "Pick a time. No obligation.",
    },
    calendlyUrl: {
        type: ControlType.String,
        title: "📞 Calendly URL",
        defaultValue: "https://calendly.com",
    },
    formTitle: {
        type: ControlType.String,
        title: "📞 Form Title",
        defaultValue: "Send a Message",
    },
    formSubmitText: {
        type: ControlType.String,
        title: "📞 Submit Text",
        defaultValue: "Send Message",
    },
    formSuccessTitle: {
        type: ControlType.String,
        title: "📞 Success Title",
        defaultValue: "Message Sent!",
    },
    formSuccessBody: {
        type: ControlType.String,
        title: "📞 Success Body",
        defaultValue: "I'll get back within 24 hours.",
    },
    contactEmail: {
        type: ControlType.String,
        title: "📞 Email",
        defaultValue: "david@aiaa.agency",
    },
    contactLocation: {
        type: ControlType.String,
        title: "📞 Location",
        defaultValue: "Los Angeles, CA",
    },
    contactResponse: {
        type: ControlType.String,
        title: "📞 Response",
        defaultValue: "Within 2-4 hours",
    },

    // ─── BLOG ───
    blogPageTitle: {
        type: ControlType.String,
        title: "📝 Title",
        defaultValue: "Insights",
    },
    blogPageSubtitle: {
        type: ControlType.String,
        title: "📝 Subtitle",
        defaultValue: "AI automation and digital strategy.",
    },
    blog1Title: {
        type: ControlType.String,
        title: "📝 B1 Title",
        defaultValue: "Why Dentists Lose $10K/Month",
    },
    blog1Date: {
        type: ControlType.String,
        title: "📝 B1 Date",
        defaultValue: "January 15, 2025",
    },
    blog1Excerpt: {
        type: ControlType.String,
        title: "📝 B1 Excerpt",
        defaultValue: "The average practice loses $8-15K to no-shows.",
    },
    blog1Image: { type: ControlType.Image, title: "📝 B1 Image" },
    blog2Title: {
        type: ControlType.String,
        title: "📝 B2 Title",
        defaultValue: "AI Chatbot in 30 Minutes",
    },
    blog2Date: {
        type: ControlType.String,
        title: "📝 B2 Date",
        defaultValue: "January 28, 2025",
    },
    blog2Excerpt: {
        type: ControlType.String,
        title: "📝 B2 Excerpt",
        defaultValue: "Production-ready chatbot in 30 min.",
    },
    blog2Image: { type: ControlType.Image, title: "📝 B2 Image" },
    blog3Title: {
        type: ControlType.String,
        title: "📝 B3 Title",
        defaultValue: "Tripled a Clinic's Google Rating",
    },
    blog3Date: {
        type: ControlType.String,
        title: "📝 B3 Date",
        defaultValue: "February 10, 2025",
    },
    blog3Excerpt: {
        type: ControlType.String,
        title: "📝 B3 Excerpt",
        defaultValue: "3.2 to 4.8 stars automatically.",
    },
    blog3Image: { type: ControlType.Image, title: "📝 B3 Image" },
    blog4Title: {
        type: ControlType.String,
        title: "📝 B4 Title",
        defaultValue: "Real Estate Agents Need More",
    },
    blog4Date: {
        type: ControlType.String,
        title: "📝 B4 Date",
        defaultValue: "February 24, 2025",
    },
    blog4Excerpt: {
        type: ControlType.String,
        title: "📝 B4 Excerpt",
        defaultValue: "Pretty doesn't convert.",
    },
    blog4Image: { type: ControlType.Image, title: "📝 B4 Image" },
    blog5Title: {
        type: ControlType.String,
        title: "📝 B5 Title",
        defaultValue: "3 Videos a Day AI Pipeline",
    },
    blog5Date: {
        type: ControlType.String,
        title: "📝 B5 Date",
        defaultValue: "March 8, 2025",
    },
    blog5Excerpt: {
        type: ControlType.String,
        title: "📝 B5 Excerpt",
        defaultValue: "The exact production system.",
    },
    blog5Image: { type: ControlType.Image, title: "📝 B5 Image" },
    blogReadMoreText: {
        type: ControlType.String,
        title: "📝 Read More",
        defaultValue: "Read More →",
    },

    // ─── PORTFOLIO PAGE ───
    portfolioPageTitle: {
        type: ControlType.String,
        title: "📂 Title",
        defaultValue: "The Work",
    },
    portfolioPageSubtitle: {
        type: ControlType.String,
        title: "📂 Subtitle",
        defaultValue: "Real systems. Real results.",
    },
    pf1Title: {
        type: ControlType.String,
        title: "📂 PF1 Title",
        defaultValue: "Classic Clinic",
    },
    pf1Desc: {
        type: ControlType.String,
        title: "📂 PF1 Desc",
        defaultValue: "Full healthcare web design.",
    },
    pf1Tags: {
        type: ControlType.String,
        title: "📂 PF1 Tags",
        defaultValue: "healthcare",
    },
    pf1Image: { type: ControlType.Image, title: "📂 PF1 Image" },
    pf2Title: {
        type: ControlType.String,
        title: "📂 PF2 Title",
        defaultValue: "Dental Clinic",
    },
    pf2Desc: {
        type: ControlType.String,
        title: "📂 PF2 Desc",
        defaultValue: "Dental practice website.",
    },
    pf2Tags: {
        type: ControlType.String,
        title: "📂 PF2 Tags",
        defaultValue: "healthcare",
    },
    pf2Image: { type: ControlType.Image, title: "📂 PF2 Image" },
    pf3Title: {
        type: ControlType.String,
        title: "📂 PF3 Title",
        defaultValue: "Dream Realty LA",
    },
    pf3Desc: {
        type: ControlType.String,
        title: "📂 PF3 Desc",
        defaultValue: "Luxury real estate platform.",
    },
    pf3Tags: {
        type: ControlType.String,
        title: "📂 PF3 Tags",
        defaultValue: "real-estate",
    },
    pf3Image: { type: ControlType.Image, title: "📂 PF3 Image" },
    pf4Title: {
        type: ControlType.String,
        title: "📂 PF4 Title",
        defaultValue: "Frank Lopez",
    },
    pf4Desc: {
        type: ControlType.String,
        title: "📂 PF4 Desc",
        defaultValue: "Agent website.",
    },
    pf4Tags: {
        type: ControlType.String,
        title: "📂 PF4 Tags",
        defaultValue: "real-estate",
    },
    pf4Image: { type: ControlType.Image, title: "📂 PF4 Image" },
    pf5Title: {
        type: ControlType.String,
        title: "📂 PF5 Title",
        defaultValue: "Alina Muradyan",
    },
    pf5Desc: {
        type: ControlType.String,
        title: "📂 PF5 Desc",
        defaultValue: "Premium web presence.",
    },
    pf5Tags: {
        type: ControlType.String,
        title: "📂 PF5 Tags",
        defaultValue: "real-estate",
    },
    pf5Image: { type: ControlType.Image, title: "📂 PF5 Image" },
    pf6Title: {
        type: ControlType.String,
        title: "📂 PF6 Title",
        defaultValue: "30-Min Chatbot",
    },
    pf6Desc: {
        type: ControlType.String,
        title: "📂 PF6 Desc",
        defaultValue: "AI chatbot in 30 min.",
    },
    pf6Tags: {
        type: ControlType.String,
        title: "📂 PF6 Tags",
        defaultValue: "ai",
    },
    pf6Image: { type: ControlType.Image, title: "📂 PF6 Image" },
    pf7Title: {
        type: ControlType.String,
        title: "📂 PF7 Title",
        defaultValue: "YouTube Pipeline",
    },
    pf7Desc: {
        type: ControlType.String,
        title: "📂 PF7 Desc",
        defaultValue: "3 videos/day automated.",
    },
    pf7Tags: {
        type: ControlType.String,
        title: "📂 PF7 Tags",
        defaultValue: "content",
    },
    pf7Image: { type: ControlType.Image, title: "📂 PF7 Image" },
    pf8Title: {
        type: ControlType.String,
        title: "📂 PF8 Title",
        defaultValue: "No-Show Prevention",
    },
    pf8Desc: {
        type: ControlType.String,
        title: "📂 PF8 Desc",
        defaultValue: "Reduced no-shows by 40%.",
    },
    pf8Tags: {
        type: ControlType.String,
        title: "📂 PF8 Tags",
        defaultValue: "ai,healthcare",
    },
    pf8Image: { type: ControlType.Image, title: "📂 PF8 Image" },

    // ─── CASE STUDIES ───
    cs1Challenge: {
        type: ControlType.String,
        title: "📋 CS1 Challenge",
        defaultValue: "Outdated website, no leads.",
    },
    cs1Solution: {
        type: ControlType.String,
        title: "📋 CS1 Solution",
        defaultValue: "Built complete digital platform.",
    },
    cs1Outcome: {
        type: ControlType.String,
        title: "📋 CS1 Outcome",
        defaultValue: "65% increase in bookings.",
    },
    cs2Challenge: {
        type: ControlType.String,
        title: "📋 CS2 Challenge",
        defaultValue: "No web presence.",
    },
    cs2Solution: {
        type: ControlType.String,
        title: "📋 CS2 Solution",
        defaultValue: "Built dental website.",
    },
    cs2Outcome: {
        type: ControlType.String,
        title: "📋 CS2 Outcome",
        defaultValue: "23 bookings in 30 days.",
    },

    // ─── FOOTER ───
    footerCopy: {
        type: ControlType.String,
        title: "🦶 Copyright",
        defaultValue: "© 2025 David — AIAA. All rights reserved.",
    },
    showFooterLinks: {
        type: ControlType.Boolean,
        title: "🦶 Show Links",
        defaultValue: true,
    },
    showFooterSocial: {
        type: ControlType.Boolean,
        title: "🦶 Show Social",
        defaultValue: true,
    },
    socialLinkedinUrl: {
        type: ControlType.String,
        title: "🦶 LinkedIn URL",
        defaultValue: "https://linkedin.com",
    },
    socialYoutubeUrl: {
        type: ControlType.String,
        title: "🦶 YouTube URL",
        defaultValue: "https://youtube.com",
    },
    socialTwitterUrl: {
        type: ControlType.String,
        title: "🦶 Twitter URL",
        defaultValue: "https://twitter.com",
    },
    socialGithubUrl: {
        type: ControlType.String,
        title: "🦶 GitHub URL",
        defaultValue: "",
    },
    socialIconSize: {
        type: ControlType.Number,
        title: "🦶 Icon Size",
        defaultValue: 36,
        min: 24,
        max: 56,
    },

    // ─── SECTION TOGGLES ───
    showSpeedCallout: {
        type: ControlType.Boolean,
        title: "🔘 Speed Section",
        defaultValue: true,
    },
    showMetrics: {
        type: ControlType.Boolean,
        title: "🔘 Metrics",
        defaultValue: true,
    },
    showPortfolioPreview: {
        type: ControlType.Boolean,
        title: "🔘 Portfolio Preview",
        defaultValue: true,
    },
    showProcess: {
        type: ControlType.Boolean,
        title: "🔘 Process",
        defaultValue: true,
    },
    showTestimonials: {
        type: ControlType.Boolean,
        title: "🔘 Testimonials",
        defaultValue: true,
    },
    showFinalCta: {
        type: ControlType.Boolean,
        title: "🔘 Final CTA",
        defaultValue: true,
    },
    showTrustStrip: {
        type: ControlType.Boolean,
        title: "🔘 Trust Strip",
        defaultValue: true,
    },
    showProblems: {
        type: ControlType.Boolean,
        title: "🔘 Problems",
        defaultValue: true,
    },
    showServicesOverview: {
        type: ControlType.Boolean,
        title: "🔘 Services",
        defaultValue: true,
    },
})
