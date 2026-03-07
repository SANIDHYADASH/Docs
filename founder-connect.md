# FounderConnect

FounderConnect is a web platform designed to connect startup founders with technical co-founders. It facilitates the process of finding the right technical partner by allowing founders to post their startup ideas and developers to browse and apply to these opportunities.

## 🚀 Features

### For Founders
- **Post Startup Ideas**: Share your startup vision with detailed descriptions
- **Manage Applications**: Review and respond to developer applications
- **Equity & Compensation**: Set clear expectations for equity shares and compensation
- **Profile Management**: Maintain your founder profile with contact information

### For Developers
- **Browse Opportunities**: Explore available startup ideas
- **Easy Application**: Apply to interesting projects with your proposal
- **Professional Profile**: Showcase your GitHub and LinkedIn profiles
- **Application Tracking**: Monitor the status of your applications

## 🛠️ Technology Stack

- **Frontend**: React.js with TypeScript
- **Styling**: Tailwind CSS
- **Authentication**: Firebase Auth
- **Database**: Firebase Firestore
- **State Management**: React Context API
- **Form Handling**: React Hook Form
- **Icons**: Lucide React
- **Build Tool**: Vite

## 🏗️ Architecture

### Components Structure
```
src/
├── components/
│   ├── Navbar.tsx
│   └── ProtectedRoute.tsx
├── contexts/
│   └── AuthContext.tsx
├── lib/
│   └── firebase.ts
├── pages/
│   ├── BrowseIdeas.tsx
│   ├── Dashboard.tsx
│   ├── Home.tsx
│   ├── Login.tsx
│   ├── PostIdea.tsx
│   ├── Profile.tsx
│   └── Register.tsx
└── types/
    └── index.ts
```

### Key Features Implementation

#### Authentication
- Email/password-based authentication using Firebase Auth
- Protected routes for authenticated users
- Role-based access control (Founder/Developer)

#### Data Models
```typescript
// User Types
type UserRole = 'founder' | 'developer';

interface User {
  uid: string;
  email: string;
  role: UserRole;
  name: string;
  githubProfile?: string;
  linkedinProfile?: string;
  whatsappNumber?: string;
}

// Startup Idea
interface StartupIdea {
  id: string;
  founderId: string;
  title: string;
  description: string;
  equityRange: string;
  salaryRange: string;
  skills: string[];
  createdAt: Date;
}

// Application
interface Application {
  id: string;
  ideaId: string;
  developerId: string;
  proposal: string;
  equityRequest: string;
  salaryRequest: string;
  status: 'pending' | 'accepted' | 'rejected';
  rejectionReason?: string;
  createdAt: Date;
}
```

## 🚀 Getting Started

### Prerequisites
- Node.js (v14 or higher)
- npm or yarn
- Firebase account

### Installation

1. Clone the repository
```bash
git clone https://github.com/sanidhyadash/FounderConnect.git
cd founder-connect
```

2. Install dependencies
```bash
npm install
```

3. Set up Firebase
- Create a new Firebase project
- Enable Authentication and Firestore
- Copy your Firebase configuration
- Update `src/lib/firebase.ts` with your configuration

4. Start the development server
```bash
npm run dev
```

## 🔒 Security

- Firebase Authentication for secure user management
- Protected routes using React Router
- Role-based access control
- Secure data access patterns in Firestore

## 🎯 Future Enhancements

1. **Chat System**
   - Implement real-time messaging between founders and developers
   - Add notification system for new messages

2. **Advanced Filtering**
   - Filter startup ideas by technology stack
   - Search functionality
   - Skill-based matching

3. **Profile Enhancements**
   - Portfolio showcase for developers
   - Previous startup experience for founders
   - Skill verification system

4. **Analytics Dashboard**
   - Application success rates
   - Popular technology stacks
   - User engagement metrics

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the project
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👨‍💻 Author

Made with ❤️ by Sanidhya

## 🙏 Acknowledgments

- [React](https://reactjs.org/)
- [Firebase](https://firebase.google.com/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Lucide Icons](https://lucide.dev/)