# 🚀 Sanidhya Dash - Portfolio Website

<div align="center">

![Portfolio Banner](https://img.shields.io/badge/Portfolio-Sanidhya%20Dash-00D9FF?style=for-the-badge)
![React](https://img.shields.io/badge/React-18.3.1-61DAFB?style=for-the-badge&logo=react)
![TypeScript](https://img.shields.io/badge/TypeScript-5.0-3178C6?style=for-the-badge&logo=typescript)
![Tailwind](https://img.shields.io/badge/Tailwind-CSS-38B2AC?style=for-the-badge&logo=tailwind-css)

**A modern, fully responsive portfolio website showcasing my journey as a Software Engineer**

[🌐 Live Demo](https://sanidhya.in) • [📧 Contact Me](mailto:sanidhyadash32@gmail.com) • [💼 LinkedIn](https://linkedin.com/in/sanidhyadash)

</div>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Technology Stack](#-technology-stack)
- [Project Architecture](#-project-architecture)
- [Design System](#-design-system)
- [Components Documentation](#-components-documentation)
- [Pages & Routing](#-pages--routing)
- [Animations & Effects](#-animations--effects)
- [Installation & Setup](#-installation--setup)
- [Deployment](#-deployment)
- [Performance Optimization](#-performance-optimization)
- [Browser Support](#-browser-support)
- [Future Enhancements](#-future-enhancements)
- [Contributing](#-contributing)
- [License](#-license)
- [Contact](#-contact)

---

## 🎯 Overview

This is a **personal portfolio website** built to showcase my skills, experience, education, and projects as a Software Engineer. The website features a modern, clean design with smooth animations, da[...]

### Key Highlights

- ✨ **Modern UI/UX**: Clean, professional design with smooth animations
- 🎨 **Dual Theme**: Dark and light mode support with seamless transitions
- 📱 **Fully Responsive**: Optimized for mobile, tablet, and desktop devices
- ⚡ **Performance Optimized**: Fast loading with lazy loading and code splitting
- 🎭 **Interactive Elements**: Engaging hover effects and scroll animations
- 📊 **Project Showcase**: Detailed documentation for featured projects
- 📧 **Contact Integration**: Direct email integration for inquiries
- 🔍 **SEO Optimized**: Proper meta tags and semantic HTML structure

---

## ✨ Features

### 🏠 Hero Section
- **Animated Typing Effect**: Dynamic text animation displaying role
- **Floating Gradient Orbs**: Three animated background elements with blur effects
- **Social Media Links**: GitHub, LinkedIn, and Email with hover effects
- **CTA Buttons**: "Get In Touch" and "Download Resume" with gradient styling
- **Scroll Indicator**: Animated scroll indicator at bottom

### 👤 About Section
- **Profile Image**: Professional photo with gradient border and blur effect
- **Highlight Cards**: Four feature cards showcasing key strengths:
  - Clean Code
  - Innovation
  - Performance
  - Collaboration
- **Intersection Observer**: Elements fade in on scroll into view
- **Gradient Overlays**: Each card has subtle gradient background

### 🛠️ Skills Section
- **Categorized Display**: Skills organized into 4 categories:
  - Frontend Development
  - Backend Development
  - Big Data & Analytics
  - Other Tools
- **Icon Display**: Technology logos from CDN with fallback handling
- **Hover Effects**: Scale and color transitions on hover
- **Gradient Cards**: Each category card has gradient overlay and enhanced borders

### 🎓 Education Section
- **Timeline Display**: Educational journey from high school to master's
- **Institution Logos**: Visual representation with fallback
- **Detailed Descriptions**: Comprehensive course information
- **Calendar Icons**: Visual date indicators
- **Gradient Overlays**: Enhanced card styling with borders

### 💼 Experience Section
- **Professional Journey**: Detailed work experience
- **Company Logos**: Professional branding display
- **Skill Tags**: Technologies used in each role with badge styling
- **Gradient Cards**: Consistent design with Skills and Education sections

### 🚀 Projects Section
- **Carousel Display**: Swipeable project cards using Embla Carousel
- **Featured Projects**:
  - **Flink**: File and text sharing platform with Supabase backend
  - **FounderConnect**: Startup founder matching platform with Firebase
  - **Musical Artist Portfolio**: Full-stack event management system
- **Quick Actions**: Direct links to GitHub and live demos
- **Project Details Pages**: Individual pages with comprehensive documentation
- **Gradient Themes**: Each project has unique gradient styling

### 📞 Contact Section
- **Contact Form**: Name, email, and message fields with validation
- **Direct Email Integration**: Opens default email client with pre-filled information
- **Contact Info Cards**: Email, chat, and location information
- **Hover Effects**: Interactive card animations
- **Toast Notifications**: User feedback using Sonner

### 🧭 Navigation
- **Fixed Header**: Sticky navigation with scroll-based styling changes
- **Theme Toggle**: Sun/Moon icons for dark/light mode switching
- **Mobile Responsive**: Hamburger menu for small screens
- **Smooth Scrolling**: Animated scroll to sections
- **Active State**: Animated underline on hover
- **Route Handling**: Support for both hash navigation and React Router

### ⚡ Loading Screen
- **Session Persistence**: Shows only once per session
- **Animated Logo**: "SD" initials with staggered animation
- **Progress Bar**: Gradient loading indicator
- **Pulse Effects**: Multiple animated elements
- **Code Snippets**: Tech-themed background decorations

---

## 🛠️ Technology Stack

### Core Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| **React** | 18.3.1 | UI library for building components |
| **TypeScript** | 5.0+ | Type-safe JavaScript |
| **Vite** | Latest | Fast build tool and dev server |
| **Tailwind CSS** | 3.4+ | Utility-first CSS framework |

### UI Libraries & Components

| Library | Purpose |
|---------|---------|
| **shadcn/ui** | Pre-built accessible UI components |
| **Radix UI** | Unstyled accessible primitives |
| **Lucide React** | Modern icon library |
| **Embla Carousel** | Touch-friendly carousel component |

### State Management & Utilities

| Package | Purpose |
|---------|---------|
| **React Router DOM** | Client-side routing |
| **React Hook Form** | Form state management |
| **TanStack Query** | Data fetching and caching |
| **next-themes** | Theme management (dark/light) |
| **Sonner** | Toast notifications |
| **Zod** | Schema validation |

### Development Tools

| Tool | Purpose |
|------|---------|
| **ESLint** | Code linting |
| **PostCSS** | CSS processing |
| **Autoprefixer** | Browser compatibility |
| **Class Variance Authority** | Component variant management |
| **clsx & tailwind-merge** | Conditional class name handling |

---

## 🏗️ Project Architecture

### Directory Structure

```
src/
├── assets/                  # Static assets (images, icons)
│   ├── profile-photo.jpeg   # Profile picture
│   └── architecture-flink.png
│
├── components/              # React components
│   ├── ui/                  # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── input.tsx
│   │   ├── carousel.tsx
│   │   └── ... (40+ components)
│   │
│   ├── About.tsx            # About section component
│   ├── Skills.tsx           # Skills section component
│   ├── Education.tsx        # Education section component
│   ├── Experience.tsx       # Experience section component
│   ├── Projects.tsx         # Projects carousel component
│   ├── Contact.tsx          # Contact form component
│   ├── Hero.tsx             # Hero section with animations
│   ├── Navigation.tsx       # Header navigation
│   ├── Footer.tsx           # Footer component
│   ├── LoadingScreen.tsx    # Initial loading animation
│   └── ProjectDocumentation.tsx  # Project detail content
│
├── data/                    # Data files
│   ├── flink-docs.tsx       # Flink project documentation
│   ├── founderconnect-docs.tsx
│   └── musical-portfolio-docs.tsx
│
├── hooks/                   # Custom React hooks
│   ├── use-mobile.tsx       # Mobile detection hook
│   └── use-toast.ts         # Toast notification hook
│
├── lib/                     # Utility functions
│   └── utils.ts             # Helper functions (cn, etc.)
│
├── pages/                   # Page components
│   ├── Index.tsx            # Home page
│   ├── AllProjects.tsx      # Projects list page
│   ├── ProjectDetail.tsx    # Individual project page
│   └── NotFound.tsx         # 404 error page
│
├── App.tsx                  # Main app component with routing
├── main.tsx                 # Application entry point
├── index.css                # Global styles & design system
└── vite-env.d.ts           # TypeScript declarations

public/
├── Sanidhya_Dash_Resume.pdf  # Resume download
└── robots.txt                # SEO configuration

Configuration Files:
├── tailwind.config.ts       # Tailwind configuration
├── vite.config.ts           # Vite build configuration
├── tsconfig.json            # TypeScript configuration
├── eslint.config.js         # ESLint rules
└── vercel.json              # Deployment configuration
```

### Component Hierarchy

```
App
├── ThemeProvider (Dark/Light Mode)
├── QueryClientProvider (Data Fetching)
├── TooltipProvider (Tooltips)
└── BrowserRouter (Routing)
    ├── LoadingScreen (Session-based)
    └── Routes
        ├── Index (Home Page)
        │   ├── Navigation
        │   ├── Hero
        │   ├── About
        │   ├── Skills
        │   ├── Education
        │   ├── Experience
        │   ├── Projects
        │   ├── Contact
        │   └── Footer
        │
        ├── AllProjects
        │   ├── Navigation
        │   ├── Project Grid
        │   └── Footer
        │
        ├── ProjectDetail
        │   ├── Navigation
        │   ├── Project Header
        │   ├── ProjectDocumentation
        │   └── Footer
        │
        └── NotFound
```

---

## 🎨 Design System

### Color Palette

The design system uses **HSL color values** for better theme management and manipulation.

#### Light Mode Colors

```css
--primary: 190 95% 55%        /* Cyan Blue - #00D9FF */
--secondary: 260 60% 55%      /* Purple - #7D5FFF */
--accent: 290 70% 60%         /* Magenta - #E066FF */
--background: 0 0% 100%       /* White */
--foreground: 222.2 84% 4.9%  /* Near Black */
--muted: 210 40% 96.1%        /* Light Gray */
--border: 214.3 31.8% 91.4%   /* Border Gray */
```

#### Dark Mode Colors

```css
--primary: 190 95% 55%        /* Cyan Blue - Same */
--secondary: 260 60% 55%      /* Purple - Same */
--accent: 290 70% 60%         /* Magenta - Same */
--background: 220 25% 6%      /* Dark Blue-Gray */
--foreground: 210 40% 98%     /* Off White */
--muted: 220 15% 20%          /* Dark Gray */
--border: 220 15% 20%         /* Dark Border */
```

### Gradient System

```css
/* Primary Gradient: Cyan to Purple */
--gradient-primary: linear-gradient(135deg, 
  hsl(190 95% 55%), 
  hsl(260 60% 55%)
);

/* Secondary Gradient: Purple to Magenta */
--gradient-secondary: linear-gradient(135deg, 
  hsl(260 60% 55%), 
  hsl(290 70% 60%)
);

/* Subtle Accent Gradient */
--gradient-accent: linear-gradient(135deg, 
  hsl(190 95% 55% / 0.1), 
  hsl(260 60% 55% / 0.1)
);
```

### Glow Effects

```css
--glow-primary: 0 0 40px hsl(190 95% 55% / 0.3);
--glow-secondary: 0 0 40px hsl(260 60% 55% / 0.3);
```

### Typography

- **Font Family**: 'Inter', sans-serif (via Google Fonts)
- **Font Weights**: 400 (regular), 500 (medium), 600 (semibold), 700 (bold)
- **Font Sizes**: Responsive using Tailwind's size scale

### Border Radius

```css
--radius: 0.75rem  /* 12px base radius */
lg: var(--radius)
md: calc(var(--radius) - 2px)
sm: calc(var(--radius) - 4px)
```

### Spacing

Following Tailwind's default spacing scale (4px base unit):
- `p-4` = 1rem (16px)
- `p-6` = 1.5rem (24px)
- `p-8` = 2rem (32px)

### Custom Scrollbar

```css
::-webkit-scrollbar {
  width: 10px;
}

::-webkit-scrollbar-track {
  background: hsl(var(--background));
}

::-webkit-scrollbar-thumb {
  background: hsl(var(--primary) / 0.3);
  border-radius: 5px;
}

::-webkit-scrollbar-thumb:hover {
  background: hsl(var(--primary) / 0.5);
}
```

---

## 📦 Components Documentation

### Core Components

#### `Hero.tsx`
**Purpose**: Landing section with animated introduction

**Features**:
- Typewriter effect for role text
- Three floating gradient orbs with staggered animations
- Social media icon links with hover effects
- CTA buttons with gradient styling
- Animated scroll indicator

**State Management**:
- `text`: Current typed text
- `isDeleting`: Typing direction flag
- `loopNum`: Loop counter
- `typingSpeed`: Dynamic typing speed (150ms normal, 50ms deleting)

**Key Animations**:
- `animate-float`: Continuous up/down motion (6s loop)
- `animate-fade-in`: Fade in with translateY (0.6s)
- `animate-pulse`: Pulsing opacity (2s loop)
- `animate-bounce`: Scroll indicator bounce

#### `About.tsx`
**Purpose**: Personal introduction with highlight cards

**Features**:
- Profile photo with gradient border
- Four highlight cards (Clean Code, Innovation, Performance, Collaboration)
- Intersection Observer for scroll animations
- Responsive grid layout

**Intersection Observer**:
```typescript
const observer = new IntersectionObserver(
  ([entry]) => {
    if (entry.isIntersecting) {
      setIsVisible(true);
    }
  },
  { threshold: 0.1 }
);
```

#### `Skills.tsx`
**Purpose**: Showcase technical skills by category

**Data Structure**:
```typescript
interface Skill {
  name: string;
  icon: string;  // CDN URL
}

interface SkillCategory {
  category: string;
  skills: Skill[];
}
```

**Categories**:
1. Frontend (7 skills)
2. Backend (9 skills)
3. Big Data & Analytics (7 skills)
4. Others (8 skills)

**Features**:
- CDN icon loading with fallback
- Gradient overlay cards
- Hover scale effects
- Staggered animations

#### `Education.tsx` & `Experience.tsx`
**Purpose**: Display academic and professional journey

**Common Features**:
- Logo display with fallback handling
- Calendar icon for dates
- Gradient overlay on cards
- Enhanced borders (2px, primary/30 → primary/60)
- Hover effects with scale and shadow

**Education Data Points**:
- IIT Jodhpur (M.Tech AI, 2024-2026)
- LPU Punjab (B.Tech CSE, 2018-2022)
- Prabhujee School (XII, 2016-2018)
- Prabhujee School (X, 2016)

**Experience Data Points**:
- FIS India (Software Engineer, June 2022 - Present)
- IBM India (Data Engineer Intern, Jan 2022 - June 2022)

#### `Projects.tsx`
**Purpose**: Showcase featured projects in carousel

**Carousel Configuration**:
```typescript
opts={{
  align: "start",
  loop: true,
  watchDrag: true,
}}
```

**Project Data Structure**:
```typescript
interface Project {
  id: string;
  title: string;
  description: string;
  tags: string[];
  gradient: string;
  codeLink: string;
  liveLink: string;
}
```

**Features**:
- Embla Carousel for smooth swiping
- Gradient overlay per project
- Click to navigate to detail page
- Direct GitHub and demo links
- Responsive card sizing

#### `Contact.tsx`
**Purpose**: Contact form and information

**Form Handling**:
```typescript
const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault();
  // Creates mailto link with form data
  window.location.href = `mailto:sanidhyadash32@gmail.com?subject=${subject}&body=${body}`;
  toast.success("Opening your email client...");
};
```

**Contact Info**:
- Email: sanidhyadash32@gmail.com
- Tawk Chat: https://tawk.to/sanidhyadash
- Location: Bengaluru, India

#### `Navigation.tsx`
**Purpose**: Site-wide navigation with theme toggle

**Features**:
- Fixed position with scroll-based styling
- Theme toggle (dark/light)
- Mobile hamburger menu
- Smooth scroll to sections
- Route-aware navigation
- Animated underline on hover

**Scroll Handler**:
```typescript
useEffect(() => {
  const handleScroll = () => {
    setIsScrolled(window.scrollY > 50);
  };
  window.addEventListener("scroll", handleScroll);
  return () => window.removeEventListener("scroll", handleScroll);
}, []);
```

#### `LoadingScreen.tsx`
**Purpose**: Initial loading animation

**Session Management**:
```typescript
// Only show once per session
const hasSeenLoading = sessionStorage.getItem('hasSeenLoading');
if (hasSeenLoading) {
  setIsLoading(false);
}
```

**Animation Timeline**:
1. "S" fades in and scales (0s)
2. "D" fades in and scales (0.2s)
3. Progress bar slides in (0-2s)
4. Dots pulse in sequence
5. Code snippets fade in
6. Entire screen fades out (2.5s)

---

## 🗺️ Pages & Routing

### Route Configuration

```typescript
<Routes>
  <Route path="/" element={<Index />} />
  <Route path="/projects" element={<AllProjects />} />
  <Route path="/projects/:projectId" element={<ProjectDetail />} />
  <Route path="*" element={<NotFound />} />
</Routes>
```

### Page Details

#### `Index.tsx` (Home Page)
**URL**: `/`

**Sections** (in order):
1. Hero - Introduction
2. About - Personal info
3. Skills - Technical abilities
4. Education - Academic background
5. Experience - Work history
6. Projects - Featured work
7. Contact - Get in touch
8. Footer - Links and copyright

#### `AllProjects.tsx`
**URL**: `/projects`

**Features**:
- Grid layout (1/2/3 columns)
- All projects displayed
- Click to view details
- Consistent card styling
- GitHub and demo links

#### `ProjectDetail.tsx`
**URL**: `/projects/:projectId`

**Supported IDs**:
- `flink`
- `founderconnect`
- `musical-artist-portfolio`

**Features**:
- Back to projects button
- Project header card with:
  - Title and description
  - Technology tags
  - GitHub and live site links
- Comprehensive documentation:
  - Overview
  - Features
  - Technical stack
  - Architecture
  - Setup instructions

**Documentation Loading**:
```typescript
const getDocumentation = () => {
  switch (projectId) {
    case "flink":
      return getFlinkDocumentation();
    case "founderconnect":
      return getFounderConnectDocumentation();
    case "musical-artist-portfolio":
      return getMusicalPortfolioDocumentation();
    default:
      return null;
  }
};
```

#### `NotFound.tsx`
**URL**: Any invalid route

**Features**:
- 404 error message
- Back to home button
- Consistent navigation

---

## 🎬 Animations & Effects

### Keyframe Animations

#### Fade Animations
```css
@keyframes fade-in {
  0% { opacity: 0; transform: translateY(20px); }
  100% { opacity: 1; transform: translateY(0); }
}

@keyframes fade-in-left {
  0% { opacity: 0; transform: translateX(-20px); }
  100% { opacity: 1; transform: translateX(0); }
}

@keyframes fade-in-right {
  0% { opacity: 0; transform: translateX(20px); }
  100% { opacity: 1; transform: translateX(0); }
}
```

#### Float Animation
```css
@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-20px); }
}
/* Duration: 6s ease-in-out infinite */
```

#### Glow Animation
```css
@keyframes glow {
  0%, 100% { box-shadow: 0 0 20px hsl(190 95% 55% / 0.3); }
  50% { box-shadow: 0 0 40px hsl(190 95% 55% / 0.6); }
}
/* Duration: 2s ease-in-out infinite */
```

#### Shimmer Animation
```css
@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}
/* Duration: 3s linear infinite */
```

#### Accordion Animations
```css
@keyframes accordion-down {
  from { height: 0; }
  to { height: var(--radix-accordion-content-height); }
}

@keyframes accordion-up {
  from { height: var(--radix-accordion-content-height); }
  to { height: 0; }
}
/* Duration: 0.2s ease-out */
```

### Transition Effects

#### Smooth Transition Variable
```css
--transition-smooth: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
```

#### Hover Effects

**Card Hover**:
```css
.card:hover {
  transform: scale(1.02);
  border-color: hsl(var(--primary) / 0.6);
  box-shadow: 0 0 30px hsl(190 95% 55% / 0.2);
}
```

**Button Hover**:
```css
.button:hover {
  box-shadow: 0 0 20px hsl(190 95% 55% / 0.3);
}
```

**Icon Hover**:
```css
.icon:hover {
  transform: scale(1.1);
  color: hsl(var(--primary));
}
```

### Staggered Animations

Used in multiple components for sequential appearance:

```typescript
// Use raw string to prevent Jekyll template parsing
// style={ { animationDelay: `${index * 100}ms`, transitionDelay: `${index * 150}ms` } }
// In actual React component, template literals work correctly
```

**Correct implementation in component files** (lines 755-760 reference):
```
Apply template literal syntax directly in .tsx files:
style={Object.assign({}, 
  { animationDelay: `${index * 100}ms` },
  { transitionDelay: `${index * 150}ms` }
)}
```

### Intersection Observer Animations

Components use Intersection Observer for scroll-triggered animations:

```typescript
const [isVisible, setIsVisible] = useState(false);
const sectionRef = useRef<HTMLElement>(null);

useEffect(() => {
  const observer = new IntersectionObserver(
    ([entry]) => {
      if (entry.isIntersecting) {
        setIsVisible(true);
      }
    },
    { threshold: 0.1 }
  );

  if (sectionRef.current) {
    observer.observe(sectionRef.current);
  }

  return () => observer.disconnect();
}, []);

// Apply based on isVisible state
className={`${ isVisible ? "opacity-100 translate-y-0" : "opacity-0 translate-y-10"}`}
```

---

## 💻 Installation & Setup

### Prerequisites

- **Node.js**: v16.0 or higher
- **npm**: v7.0 or higher (or yarn/pnpm)
- **Git**: For cloning the repository

### Step-by-Step Installation

1. **Clone the Repository**
```bash
git clone https://github.com/SANIDHYADASH/My-Portfolio.git
```

2. **Install Dependencies**
```bash
npm install
# or
yarn install
# or
pnpm install
```

3. **Start Development Server**
```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:5173`

### Development Scripts

```bash
# Start development server with hot reload
npm run dev

# Build for production
npm run build

# Preview production build locally
npm run preview

# Run linter
npm run lint

# Type check
npm run type-check
```

### Environment Variables

This project doesn't require environment variables for basic functionality. However, if you want to add analytics or other integrations:

Create `.env` file in root:
```env
VITE_ANALYTICS_ID=your_analytics_id
VITE_API_URL=your_api_url
```

Access in code:
```typescript
const analyticsId = import.meta.env.VITE_ANALYTICS_ID;
```

---

## 🚀 Deployment

### Deploy to Vercel (Recommended)

1. **Via Vercel CLI**
```bash
npm install -g vercel
vercel
```

2. **Via GitHub Integration**
   - Push code to GitHub
   - Import repository in Vercel dashboard
   - Auto-deploy on push to main branch

### Deploy to Netlify

1. **Build Settings**:
   - Build command: `npm run build`
   - Publish directory: `dist`

2. **netlify.toml** (optional):
```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Deploy to GitHub Pages

1. **Update vite.config.ts**:
```typescript
export default defineConfig({
  base: '/your-repo-name/',
  // ... rest of config
});
```

2. **Install gh-pages**:
```bash
npm install -D gh-pages
```

3. **Add deploy script to package.json**:
```json
{
  "scripts": {
    "predeploy": "npm run build",
    "deploy": "gh-pages -d dist"
  }
}
```

4. **Deploy**:
```bash
npm run deploy
```

### Custom Domain Setup

#### Vercel
1. Go to Project Settings → Domains
2. Add your domain
3. Configure DNS records as shown
4. Wait for SSL certificate provisioning

#### DNS Configuration
```
Type: CNAME
Name: www
Value: your-site.vercel.app
```

---

## ⚡ Performance Optimization

### Implemented Optimizations

#### 1. Code Splitting
- React Router lazy loading for routes
- Dynamic imports for heavy components

#### 2. Image Optimization
- WebP format where supported
- Lazy loading for images
- Proper sizing and srcset

#### 3. CSS Optimization
- Tailwind CSS purging unused styles
- Critical CSS inlined
- PostCSS optimization

#### 4. JavaScript Optimization
- Tree shaking via Vite
- Minification in production
- Modern bundle targeting

#### 5. Caching Strategy
- sessionStorage for loading screen state
- Service worker ready (PWA-ready)

#### 6. Bundle Analysis

```bash
npm run build
```

Check `dist/` folder size and analyze:
```bash
npx vite-bundle-visualizer
```

### Performance Metrics

**Lighthouse Scores** (Target):
- Performance: 95+
- Accessibility: 100
- Best Practices: 100
- SEO: 100

### Loading Performance

- **First Contentful Paint (FCP)**: < 1.5s
- **Largest Contentful Paint (LCP)**: < 2.5s
- **Time to Interactive (TTI)**: < 3.5s
- **Total Blocking Time (TBT)**: < 200ms
- **Cumulative Layout Shift (CLS)**: < 0.1

---

## 🌐 Browser Support

### Supported Browsers

| Browser | Version |
|---------|---------|
| Chrome | Last 2 versions |
| Firefox | Last 2 versions |
| Safari | Last 2 versions |
| Edge | Last 2 versions |
| Opera | Last 2 versions |

### Mobile Browsers

| Browser | Version |
|---------|---------|
| Chrome Android | Last 2 versions |
| Safari iOS | Last 2 versions |
| Samsung Internet | Last 2 versions |

### Feature Detection

Key features used:
- ✅ CSS Grid
- ✅ Flexbox
- ✅ CSS Custom Properties
- ✅ Intersection Observer API
- ✅ ES6+ JavaScript
- ✅ CSS Backdrop Filter

### Polyfills

Vite automatically includes necessary polyfills for:
- Promise
- fetch
- IntersectionObserver (if needed)

---

## 🔮 Future Enhancements

### Planned Features

1. **Blog Section**
   - Technical articles
   - Project case studies
   - Tutorial content

2. **Project Filtering**
   - Filter by technology
   - Filter by category
   - Search functionality

3. **Testimonials Section**
   - Client feedback
   - Colleague recommendations
   - Animated slider

4. **Analytics Dashboard**
   - Visitor statistics
   - Project view counts
   - Contact form submissions

5. **CMS Integration**
   - Easy content updates
   - Blog post management
   - Project updates

6. **Internationalization (i18n)**
   - Multi-language support
   - Language switcher
   - Localized content

7. **Enhanced Animations**
   - GSAP integration
   - Scroll-triggered animations
   - 3D effects

8. **Progressive Web App (PWA)**
   - Offline support
   - Install prompt
   - Push notifications

9. **Backend Integration**
   - Contact form to database
   - Newsletter subscription
   - Admin dashboard

10. **Accessibility Improvements**
    - Screen reader optimization
    - Keyboard navigation enhancements
    - ARIA labels refinement

---

## 🤝 Contributing

This is a personal portfolio project, but suggestions and bug reports are welcome!

### How to Contribute

1. **Fork the Repository**
2. **Create a Feature Branch**
   ```bash
   git checkout -b feature/AmazingFeature
   ```
3. **Commit Your Changes**
   ```bash
   git commit -m 'Add some AmazingFeature'
   ```
4. **Push to the Branch**
   ```bash
   git push origin feature/AmazingFeature
   ```
5. **Open a Pull Request**

### Coding Standards

- Use TypeScript for type safety
- Follow ESLint rules
- Use Prettier for formatting
- Write semantic HTML
- Use Tailwind utility classes
- Comment complex logic
- Keep components small and focused

### Commit Message Format

```
type(scope): subject

body

footer
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

**Example**:
```
feat(projects): add filter functionality

- Add filter buttons for technologies
- Implement filter logic
- Update UI to show filtered results

Closes #123
```

---

## 📄 License

This project is **open source** and available under the [MIT License](LICENSE).

```
MIT License

Copyright (c) 2024 Sanidhya Dash

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 📞 Contact

**Sanidhya Dash**

- 📧 Email: [sanidhyadash32@gmail.com](mailto:sanidhyadash32@gmail.com)
- 💼 LinkedIn: [linkedin.com/in/sanidhyadash](https://linkedin.com/in/sanidhyadash)
- 🐙 GitHub: [github.com/SANIDHYADASH](https://github.com/SANIDHYADASH)
- 💬 Tawk: [tawk.to/sanidhyadash](https://tawk.to/sanidhyadash)
- 📍 Location: Bengaluru, India

---

## 🙏 Acknowledgments

### Technologies & Libraries

- [React](https://reactjs.org/) - UI library
- [Vite](https://vitejs.dev/) - Build tool
- [Tailwind CSS](https://tailwindcss.com/) - CSS framework
- [shadcn/ui](https://ui.shadcn.com/) - Component library
- [Radix UI](https://www.radix-ui.com/) - Accessible primitives
- [Lucide](https://lucide.dev/) - Icon library
- [Embla Carousel](https://www.embla-carousel.com/) - Carousel component

---

## 📊 Project Stats

![GitHub repo size](https://img.shields.io/github/repo-size/SANIDHYADASH/portfolio)
![GitHub language count](https://img.shields.io/github/languages/count/SANIDHYADASH/portfolio)
![GitHub top language](https://img.shields.io/github/languages/top/SANIDHYADASH/portfolio)
![GitHub last commit](https://img.shields.io/github/last-commit/SANIDHYADASH/portfolio)

---

<div align="center">

### ⭐ Star this repository if you found it helpful!

**Made with ❤️ by [Sanidhya Dash](https://sanidhya.in)**

</div>
