# Event Managament Platform

A comprehensive full-stack web application for event management for artists to showcase their work, manage events, and handle ticket sales with advanced QR code verification system.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![React](https://img.shields.io/badge/React-18.3.1-61DAFB.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.8.3-3178C6.svg)
![Supabase](https://img.shields.io/badge/Supabase-Backend-3ECF8E.svg)

## 🌟 Project Overview

This platform serves as both an artist portfolio website and a comprehensive event management system. Built for musicians, performers, and event organizers, it provides everything needed to showcase artistic work and manage live performances from booking to verification. Specially designed for Aliva Maharana.

## 📋 Table of Contents

- [Features](#-features)
- [Technology Stack](#-technology-stack)
- [Architecture](#-architecture)
- [System Flow Diagrams](#-system-flow-diagrams)
- [Database Schema](#-database-schema)
- [Installation & Setup](#-installation--setup)
- [Configuration](#-configuration)
- [Usage Guide](#-usage-guide)
- [API Documentation](#-api-documentation)
- [Security Features](#-security-features)
- [Deployment](#-deployment)
- [Contributing](#-contributing)
- [License](#-license)

## 🚀 Features

### 🎨 Artist Portfolio
- **Hero Section**: Dynamic landing page with animated elements
- **About Section**: Artist biography and philosophy showcase
- **Social Media Integration**: YouTube Shorts feed display
- **Contact Management**: Direct contact form with Tawk.to live chat
- **Responsive Design**: Mobile-first approach with elegant animations

### 🎫 Event Management System
- **Show Creation & Management**: Create, edit, and manage live performances
- **Dynamic Show Display**: Automatic categorization of upcoming vs past shows
- **Venue Information**: Complete venue details with location mapping
- **Capacity Management**: Real-time ticket availability tracking
- **Image Upload**: Show poster/image management

### 💳 Advanced Ticket Booking
- **Real-time Booking**: Live ticket availability updates
- **Customer Information**: Comprehensive booking details collection
- **Price Calculation**: Dynamic pricing with tax and fee calculation
- **PDF Ticket Generation**: Automatic high-quality ticket PDF creation
- **QR Code Integration**: Embedded QR codes for seamless verification

### ✅ Ticket Verification System
- **Dual Verification Methods**:
  - QR Code Scanner (camera-based)
  - Manual verification code entry
- **Real-time Status Updates**: Instant verification results
- **Anti-fraud Protection**: Prevents double-scanning and fake tickets
- **Booking Status Tracking**: Complete audit trail of ticket usage

### 👑 Admin Dashboard
- **Multi-tab Interface**: 
  - YouTube Shorts Management
  - Announcements System
  - Show Management
  - Booking Management
- **Role-based Access Control**: Admin-only sections with authentication
- **Real-time Data**: Live updates of bookings and show statistics

### 🔐 Authentication & Security
- **Supabase Authentication**: Secure email/password authentication
- **Role-based Permissions**: User and admin role separation
- **Session Management**: Persistent login with automatic token refresh
- **Protected Routes**: Route-level security implementation

### 📱 Modern UI/UX
- **shadcn/ui Components**: Modern, accessible component library
- **Dark/Light Mode**: Theme switching capability
- **Loading Animations**: Smooth loading states and transitions
- **Toast Notifications**: Real-time user feedback system
- **Responsive Design**: Mobile-optimized layouts

## 🛠 Technology Stack

### Frontend
- **React 18.3.1**: Modern React with hooks and concurrent features
- **TypeScript 5.8.3**: Type-safe development environment
- **Vite 5.4.19**: Lightning-fast build tool and dev server
- **Tailwind CSS 3.4.17**: Utility-first CSS framework
- **shadcn/ui**: Modern React component library
- **Lucide React**: Beautiful, customizable icons

### Backend & Database
- **Supabase**: 
  - PostgreSQL database
  - Authentication service
  - Real-time subscriptions
  - Row Level Security (RLS)
  - Storage for file uploads

### Libraries & Tools
- **React Router DOM 6.30.1**: Client-side routing
- **React Query 5.83.0**: Server state management
- **React Hook Form 7.63.0**: Form handling and validation
- **date-fns 2.30.0**: Date manipulation and formatting
- **jsPDF 3.0.3**: PDF generation
- **qrcode 1.5.4**: QR code generation
- **react-qr-reader 3.0.0**: QR code scanning
- **Zod 4.1.11**: Runtime type validation

### Development Tools
- **ESLint**: Code linting and style enforcement
- **TypeScript ESLint**: TypeScript-specific linting rules
- **Autoprefixer**: CSS vendor prefixing
- **PostCSS**: CSS processing

## 🏗 Architecture

### Component Architecture

```
src/
├── components/
│   ├── ui/                    # Reusable UI components (shadcn/ui)
│   ├── admin/                 # Admin-specific components
│   ├── Header.tsx             # Navigation header
│   ├── Hero.tsx               # Landing page hero section
│   ├── Shows.tsx              # Show display component
│   ├── Contact.tsx            # Contact form
│   └── ...                    # Other feature components
├── pages/                     # Route-level page components
├── hooks/                     # Custom React hooks
├── lib/                       # Utility functions and services
├── integrations/              # External service integrations
└── types/                     # TypeScript type definitions
```

### Data Flow Architecture

The application follows a unidirectional data flow pattern:

1. **State Management**: React Query for server state + React hooks for local state
2. **API Layer**: Supabase client with TypeScript types
3. **Authentication**: Context-based auth state management
4. **Real-time Updates**: Supabase real-time subscriptions

## 📊 System Flow Diagrams


### User Ticket Booking Flow

```
User Ticket Booking Flow:

1. User visits website
2. Browse Shows
3. Is Show Available?
   ├─ Yes → Select Show
   │    ├─ Fill Booking Form
   │    ├─ Calculate Total Price
   │    ├─ Submit Booking
   │    ├─ Generate Booking ID
   │    ├─ Create QR Code
   │    ├─ Generate PDF Ticket
   │    ├─ Update Available Tickets
   │    ├─ Send Confirmation
   │    └─ Auto-download PDF
   └─ No → Show 'Sold Out'
```


### Ticket Verification Flow

```
Ticket Verification Flow:

1. Staff opens verification page
2. Choose verification method:
   ├─ QR Scanner → Scan QR Code
   │    └─ Parse QR Data
   └─ Manual Entry → Enter Code Manually
      └─ Parse QR Data
3. Query Database
4. Is Ticket Found?
   ├─ No → Display 'Invalid Ticket'
   └─ Yes → Is Already Verified?
       ├─ Yes → Display 'Already Scanned'
       └─ No → Mark as Verified
              ├─ Update Database
              ├─ Display 'Entry Granted'
              └─ Show Ticket Details
```


### Admin Management Flow

```
Admin Management Flow:

1. Admin Login
2. Verify Admin Role
3. Is Admin?
   ├─ No → Redirect to Home
   └─ Yes → Access Admin Dashboard
       ├─ Select Management Tab
       ├─ Which Tab?
       │    ├─ Shows → Manage Shows → CRUD Operations on Shows
       │    ├─ Bookings → View/Manage Bookings → View Booking Statistics
       │    ├─ Announcements → Create/Edit Announcements → Publish/Update Announcements
       │    └─ YouTube → Manage Social Feed → Update YouTube Content
```


### Authentication & Authorization Flow

```
Authentication & Authorization Flow:

1. User → Frontend: Login Request
2. Frontend → Supabase: Auth Request
3. Supabase → Database: Validate Credentials
4. Database → Supabase: User Data + Role
5. Supabase → Frontend: JWT Token + Session
6. Frontend: Store Auth State
7. Frontend: Route Protection Check
8. Is Admin Route?
   ├─ Yes → Frontend → Database: Check Admin Role
   │         Database → Frontend: Role Verification
   │         Frontend → User: Grant/Deny Access
   └─ No → Frontend → User: Grant Access
```


## 🗃 Database Schema

![Database Schema](./am-database-schema.svg)

### Core Tables

#### `shows` Table
```sql
CREATE TABLE shows (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR NOT NULL,
    venue VARCHAR NOT NULL,
    location VARCHAR NOT NULL,
    show_date DATE NOT NULL,
    show_time TIME NOT NULL,
    ticket_price DECIMAL(10,2) NOT NULL,
    available_tickets INTEGER NOT NULL,
    max_capacity INTEGER NOT NULL,
    status VARCHAR DEFAULT 'upcoming',
    image_url TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `bookings` Table
```sql
CREATE TABLE bookings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    show_id UUID REFERENCES shows(id),
    customer_name VARCHAR NOT NULL,
    customer_email VARCHAR NOT NULL,
    customer_phone VARCHAR,
    tickets_count INTEGER NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    verification_code VARCHAR(6) NOT NULL,
    booking_status VARCHAR DEFAULT 'confirmed',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `profiles` Table
```sql
CREATE TABLE profiles (
    id UUID PRIMARY KEY REFERENCES auth.users(id),
    user_id UUID REFERENCES auth.users(id),
    full_name VARCHAR,
    role VARCHAR DEFAULT 'user',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `announcements` Table
```sql
CREATE TABLE announcements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR NOT NULL,
    content TEXT NOT NULL,
    priority VARCHAR DEFAULT 'medium',
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `youtube_videos` Table
```sql
CREATE TABLE youtube_videos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    video_id VARCHAR NOT NULL,
    title VARCHAR NOT NULL,
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```


### Database Relationships

```
Database Relationships:

USERS 1--* PROFILES : has
SHOWS 1--* BOOKINGS : books for

PROFILES:
   - uuid id PK
   - uuid user_id FK
   - varchar full_name
   - varchar role
   - timestamp created_at
   - timestamp updated_at

SHOWS:
   - uuid id PK
   - varchar title
   - varchar venue
   - varchar location
   - date show_date
   - time show_time
   - decimal ticket_price
   - integer available_tickets
   - integer max_capacity
   - varchar status
   - text image_url
   - timestamp created_at
   - timestamp updated_at

BOOKINGS:
   - uuid id PK
   - uuid show_id FK
   - varchar customer_name
   - varchar customer_email
   - varchar customer_phone
   - integer tickets_count
   - decimal total_amount
   - varchar verification_code
   - varchar booking_status
   - timestamp created_at
   - timestamp updated_at

ANNOUNCEMENTS:
   - uuid id PK
   - varchar title
   - text content
   - varchar priority
   - boolean is_active
   - timestamp created_at
   - timestamp updated_at

YOUTUBE_VIDEOS:
   - uuid id PK
   - varchar video_id
   - varchar title
   - text description
   - boolean is_active
   - timestamp created_at
   - timestamp updated_at
```

## 🔧 Installation & Setup

### Prerequisites

- Node.js 18+ and npm
- Supabase account
- Git

### Quick Start

1. **Clone the Repository**
   ```bash
   git clone <repository-url>
   cd aliva-artist-platform
   ```

2. **Install Dependencies**
   ```bash
   npm install
   ```

3. **Environment Setup**
   Create a `.env.local` file:
   ```env
   VITE_SUPABASE_URL=your_supabase_url
   VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
   ```

4. **Database Setup**
   - Create a new Supabase project
   - Run the provided SQL migrations
   - Enable Row Level Security (RLS)
   - Set up authentication providers

5. **Start Development Server**
   ```bash
   npm run dev
   ```

6. **Build for Production**
   ```bash
   npm run build
   ```

### Detailed Setup Guide

#### Supabase Configuration

1. **Create Supabase Project**
   - Visit [Supabase Dashboard](https://supabase.com)
   - Create new project
   - Copy URL and API keys

2. **Database Schema Setup**
   ```sql
   -- Enable necessary extensions
   CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
   
   -- Run table creation scripts
   -- (See Database Schema section for complete SQL)
   ```

3. **Row Level Security (RLS) Policies**
   ```sql
   -- Enable RLS on all tables
   ALTER TABLE shows ENABLE ROW LEVEL SECURITY;
   ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;
   ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
   
   -- Create policies for appropriate access control
   ```

4. **Authentication Setup**
   - Enable email authentication
   - Configure email templates
   - Set up redirect URLs

## ⚙️ Configuration

### Environment Variables

```env
# Supabase Configuration
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key

# Optional: Custom Configuration
VITE_APP_TITLE=Aliva Artist Platform
VITE_CONTACT_EMAIL=sanidhyadash32@gmail.com
```

### Customization Options

#### Styling & Branding
- Update `src/index.css` for custom CSS variables
- Modify `tailwind.config.ts` for theme customization
- Replace logo and images in `public/` directory

#### Feature Toggles
- Enable/disable modules in component imports
- Customize admin dashboard tabs
- Configure payment integration (future enhancement)

## 📖 Usage Guide

### For End Users

1. **Browsing Shows**
   - Visit the homepage
   - Navigate to the Shows section
   - View upcoming and past performances

2. **Booking Tickets**
   - Click "Book Tickets" on desired show
   - Fill in personal information
   - Select number of tickets
   - Complete booking process
   - Download PDF ticket automatically

3. **Ticket Usage**
   - Present QR code or verification code at venue
   - Staff will scan/verify for entry

### For Administrators

1. **Accessing Admin Panel**
   - Login with admin credentials
   - Navigate to `/admin` route
   - Access restricted to admin role users

2. **Managing Shows**
   - Create new shows with all details
   - Upload show images
   - Monitor ticket sales
   - Update show information

3. **Managing Bookings**
   - View all bookings
   - Track ticket verification status
   - Generate reports and analytics

4. **Content Management**
   - Update announcements
   - Manage YouTube video feed
   - Handle customer communications

### For Event Staff

1. **Ticket Verification**
   - Navigate to `/verify-ticket` page
   - Use QR scanner or manual entry
   - Verify customer identity
   - Grant/deny entry based on results

## 🔌 API Documentation

### Supabase Integration

The application uses Supabase client-side SDK for all data operations:

```typescript
// Example: Fetching shows
const { data: shows, error } = await supabase
  .from('shows')
  .select('*')
  .eq('status', 'upcoming')
  .order('show_date', { ascending: true });

// Example: Creating booking
const { data: booking, error } = await supabase
  .from('bookings')
  .insert({
    show_id: showId,
    customer_name: formData.customer_name,
    // ... other fields
  })
  .select()
  .single();
```

### Key Service Functions

#### Verification Service (`src/lib/verificationService.ts`)
- `findBookingByVerificationCode(code: string)`
- `findBookingByLast6(code: string)`
- `markBookingVerified(id: string)`
- `getShowById(showId: string)`

#### PDF Ticket Generator (`src/lib/pdfTicketGenerator.ts`)
- `generateTicketPDF(ticketData: TicketData)`
- `downloadTicketPDF(ticketData: TicketData)`
- `generateVerificationCode()`

## 🔒 Security Features

### Authentication Security
- **JWT Token Management**: Automatic token refresh and secure storage
- **Role-Based Access Control**: Admin vs user permissions
- **Protected Routes**: Client-side route protection
- **Session Persistence**: Secure session management

### Data Security
- **Row Level Security (RLS)**: Database-level access control
- **Input Validation**: Zod schema validation on all forms
- **SQL Injection Prevention**: Parameterized queries via Supabase
- **XSS Protection**: React's built-in XSS prevention

### Ticket Security
- **Unique Verification Codes**: 6-digit random codes per booking
- **QR Code Encryption**: JSON payload with structured data
- **Anti-Replay Protection**: One-time verification system
- **Booking Status Tracking**: Prevents duplicate entries

### Best Practices Implemented
- HTTPS enforcement in production
- Environment variable protection
- Secure headers configuration
- Input sanitization
- Error handling without information disclosure

## 🚀 Deployment

### Vercel Deployment (Recommended)

1. **Connect Repository**
   ```bash
   # Install Vercel CLI
   npm i -g vercel
   
   # Deploy
   vercel --prod
   ```

2. **Environment Variables**
   - Add environment variables in Vercel dashboard
   - Ensure all VITE_ prefixed variables are set

3. **Build Configuration**
   ```json
   {
     "buildCommand": "npm run build",
     "outputDirectory": "dist",
     "framework": "vite"
   }
   ```

### Netlify Deployment

1. **Build Settings**
   - Build command: `npm run build`
   - Publish directory: `dist`

2. **Environment Variables**
   - Add in Netlify dashboard under Site Settings

### Self-Hosted Deployment

1. **Build Production Version**
   ```bash
   npm run build
   ```

2. **Serve Static Files**
   ```bash
   # Using serve
   npx serve -s dist
   
   # Using nginx
   # Configure nginx to serve from dist/ directory
   ```

## 🤝 Contributing

### Development Setup

1. **Fork the repository**
2. **Create feature branch**
   ```bash
   git checkout -b feature/amazing-feature
   ```
3. **Make changes and commit**
   ```bash
   git commit -m 'Add amazing feature'
   ```
4. **Push to branch**
   ```bash
   git push origin feature/amazing-feature
   ```
5. **Open Pull Request**

### Code Standards

- **TypeScript**: Strict type checking enabled
- **ESLint**: Follow configured linting rules
- **Prettier**: Code formatting (if configured)
- **Component Structure**: Use functional components with hooks
- **File Naming**: PascalCase for components, camelCase for utilities

### Testing Guidelines

- Write unit tests for utility functions
- Test component behavior with user interactions
- Verify API integration points
- Test authentication flows

## 🔮 Future Enhancements

### Planned Features

1. **Payment Integration**
   - Stripe/PayPal integration
   - Online payment processing
   - Refund management

2. **Advanced Analytics**
   - Sales reporting dashboard
   - Customer analytics
   - Performance metrics

3. **Mobile Application**
   - React Native mobile app
   - Push notifications
   - Offline ticket storage

4. **Enhanced Social Features**
   - User reviews and ratings
   - Social media sharing
   - Community features

5. **Multi-tenant Support**
   - Multiple artist support
   - Organization management
   - White-label solutions

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Supabase Team** - For the excellent backend-as-a-service platform
- **shadcn/ui** - For the beautiful component library
- **React Community** - For the robust ecosystem
- **Tailwind CSS** - For the utility-first CSS framework
- **Vercel** - For seamless deployment solutions

## 📞 Support

For support, email [sanidhyadash32@gmail.com] or create an issue in the GitHub repository.

---

**Built with ❤️ By Sanidhya Dash**

---

*This platform empowers artists to connect with their audience and manage live performances efficiently while providing fans with a seamless ticket booking and verification experience.*