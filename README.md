# Unspeciated™ Research Portal

Authentication and usage tracking gateway for the Unspeciated Research Suite.

## Overview

This portal provides:
- Google Authentication via Firebase
- User tier management (Free, Supporter, Institute)
- Usage tracking and limits
- Access control for research tools

## Structure

```
unspeciated-portal/
├── index.html       - Landing page with Google sign-in
├── dashboard.html   - User dashboard after authentication
└── README.md        - This file
```

## Setup

### 1. Firebase Configuration

The portal is already configured to use the `redefining-intelligence-suite` Firebase project.

**Required Firebase Services:**
- Authentication (Google provider enabled)
- Firestore Database

### 2. Firestore Security Rules

Add these security rules in Firebase Console → Firestore Database → Rules:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can read and write their own data
    match /users/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

### 3. Deploy to Netlify

1. Push this repository to GitHub
2. Connect to Netlify:
   - Go to https://app.netlify.com
   - Click "Add new site" → "Import an existing project"
   - Choose GitHub and select this repository
   - Build settings: Leave empty (static site, no build needed)
   - Click "Deploy site"

3. Configure custom domain:
   - In Netlify site settings → "Domain management"
   - Add custom domain: `portal.unspeciated.com`
   - Follow DNS configuration instructions

## User Tier Structure

### Free Tier
- 25 searches per month
- Access to:
  - Research Explorer
  - Researchers' Mode

### Supporter Tier
- 100 searches per month
- Access to:
  - Research Explorer
  - Researchers' Mode
  - Intelligence Map
  - AI Consciousness Bridge

### Institute Partner Tier
- Unlimited searches
- Access to all tools including:
  - Legal Assistant
  - Future premium features

## Upgrading Users

To manually upgrade a user's tier:

1. Go to Firebase Console → Firestore Database
2. Navigate to `users/[userId]`
3. Update the following fields:
   ```
   tier: "supporter" or "institute"
   monthlyLimit: 100 or -1 (for unlimited)
   features: {
     researchExplorer: true,
     researchersMode: true,
     intelligenceMap: true,
     aiConsciousnessBridge: true,
     legalAssistant: true (for institute only)
   }
   ```

## Integration with Research Tools

Each research tool will need to:
1. Check if user is authenticated
2. Increment search counter in Firestore
3. Check against tier limits before processing

Example code to add to each tool:

```javascript
// Check authentication
firebase.auth().onAuthStateChanged(async (user) => {
  if (!user) {
    // Redirect to portal
    window.location.href = 'https://portal.unspeciated.com';
    return;
  }
  
  // Get user data
  const userDoc = await db.collection('users').doc(user.uid).get();
  const userData = userDoc.data();
  
  // Check if tool is available for user's tier
  if (!userData.features.researchExplorer) {
    alert('This tool requires a higher tier. Please upgrade.');
    window.location.href = 'https://portal.unspeciated.com/dashboard.html';
    return;
  }
  
  // Check usage limit
  if (userData.monthlyLimit !== -1 && userData.searchesThisMonth >= userData.monthlyLimit) {
    alert('Monthly search limit reached. Please upgrade or wait until next month.');
    return;
  }
  
  // Tool is ready to use
  loadTool();
});

// Increment search counter after successful search
async function trackSearch(userId) {
  const userRef = db.collection('users').doc(userId);
  await userRef.update({
    searchesThisMonth: firebase.firestore.FieldValue.increment(1)
  });
}
```

## Monthly Reset

Add a Firebase Cloud Function to reset search counters on the 1st of each month:

```javascript
exports.resetMonthlySearches = functions.pubsub
  .schedule('0 0 1 * *')  // Run at midnight on the 1st of each month
  .timeZone('America/Los_Angeles')
  .onRun(async (context) => {
    const usersRef = db.collection('users');
    const snapshot = await usersRef.get();
    
    const batch = db.batch();
    const nextMonth = new Date();
    nextMonth.setMonth(nextMonth.getMonth() + 1);
    const resetDate = new Date(nextMonth.getFullYear(), nextMonth.getMonth(), 1);
    
    snapshot.docs.forEach(doc => {
      batch.update(doc.ref, {
        searchesThisMonth: 0,
        resetDate: resetDate.toISOString().split('T')[0]
      });
    });
    
    await batch.commit();
    console.log('Monthly search counters reset');
  });
```

## Support

For questions or issues:
- Email: kerri@generateharmony.com
- Website: https://generateharmony.com

## License

© 2026 Generation of Harmony LLC | Intuitive Learning Foundation
