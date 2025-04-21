# Security To-Do for Terms of Use

## Environment Variables Setup

This file outlines the environment variables that need to be set up for secure deployment of the Terms of Use application.

### Required Environment Variables

The following environment variables should be configured:

- **JWT_SECRET**: Secret key used for signing JWT tokens
  - Currently hardcoded as `"devias-top-secret-key"` in `/src/utils/jwt.js`
  - **CRITICAL**: This is a high-priority security issue

- **DEMO_USER_EMAIL**: Demo user email for testing
  - Currently hardcoded as `"onelawus@gmail.com"` in `/src/api/auth/data.js`

- **DEMO_USER_PASSWORD**: Demo user password for testing
  - Currently hardcoded as `"pw123xyz"` in `/src/api/auth/data.js`

### Implementation Steps

1. Create a `.env.local` file in the project root with the following template:
   ```
   # Authentication
   JWT_SECRET=use_a_strong_random_string_here
   
   # Demo User (for development only)
   DEMO_USER_EMAIL=example@example.com
   DEMO_USER_PASSWORD=use_a_strong_password_here
   ```

2. Update the JWT implementation in `/src/utils/jwt.js`:
   ```javascript
   // Replace this line:
   const JWT_SECRET = 'devias-top-secret-key';
   
   // With:
   const JWT_SECRET = process.env.JWT_SECRET || 'development-jwt-secret';
   
   // Add validation:
   if (process.env.NODE_ENV === 'production' && !process.env.JWT_SECRET) {
     console.error('WARNING: JWT_SECRET environment variable not set in production!');
   }
   ```

3. Update the demo user in `/src/api/auth/data.js`:
   ```javascript
   // Replace this:
   export const users = [
     {
       id: '5e86809283e28b96d2d38537',
       avatar: '/assets/avatars/avatar-scott-10.jpg',
       email: 'onelawus@gmail.com',
       name: 'Scott Evidence',
       password: 'pw123xyz',
       plan: 'Premium'
     }
   ];
   
   // With:
   export const users = [
     {
       id: '5e86809283e28b96d2d38537',
       avatar: '/assets/avatars/avatar-scott-10.jpg',
       email: process.env.DEMO_USER_EMAIL || 'demo@example.com',
       name: 'Scott Evidence',
       password: process.env.DEMO_USER_PASSWORD || 'demo_only_123',
       plan: 'Premium'
     }
   ];
   ```

4. For client-side access, update `next.config.js` to expose specific variables to the browser:
   ```javascript
   module.exports = {
     // Existing configuration
     // ...
     
     // Add environment variables to be exposed to the browser
     env: {
       // Note: Do NOT expose JWT_SECRET to the browser
       DEMO_USER_EMAIL: process.env.DEMO_USER_EMAIL,
       // For demo purposes only - in production you wouldn't expose this
       DEMO_USER_PASSWORD: process.env.NODE_ENV === 'development' ? process.env.DEMO_USER_PASSWORD : '',
     },
   };
   ```

5. Add the `.env*` files to `.gitignore` to prevent committing secrets:
   ```
   # .gitignore
   .env
   .env.local
   .env.development
   .env.production
   ```

6. For production deployment, set these environment variables in Vercel:
   - Go to your project in the Vercel dashboard
   - Navigate to Settings → Environment Variables
   - Add the variables JWT_SECRET with a strong random value
   - Add DEMO_USER_EMAIL and DEMO_USER_PASSWORD if testing accounts are needed
   - Redeploy your application

### Additional Security Enhancements

1. **JWT Implementation Improvements**:
   - Update the JWT implementation to include proper expiration (exp claim)
   - Add token rotation/refresh functionality
   - Consider using a library like `next-auth` for more secure authentication

2. **Credential Management**:
   - Replace the mock user store with a proper database in production
   - Ensure passwords are properly hashed (e.g., using bcrypt)
   - Implement proper password reset flows

3. **Demo User Security**:
   - Create a separate mechanism for demo access that doesn't rely on hardcoded credentials
   - Consider implementing a "demo mode" toggle that uses ephemeral credentials

4. **Secret Rotation**:
   - Periodically rotate the JWT_SECRET in production
   - Implement a strategy for invalidating existing tokens when the secret changes

### Security Best Practices

- Never commit sensitive credentials to version control
- Use strong, randomly generated secrets for JWT tokens
- Implement proper password hashing and storage
- Store user credentials in a secure database, not in code
- Set appropriate security headers (already configured in vercel.json)
- Consider using a dedicated authentication service for production