# GlobeWire - Testing Guide

Step-by-step manual tests for every feature, originally written for markers. See the main
[README](../README.md) for setup.

## Test accounts

### 🛒 Customer Account (For Creating Payments)

| Field | Value |
|-------|-------|
| **Username** | `testcustomer` |
| **Account Number** | `12345678` |
| **Password** | `TestPass123!` |
| **Login URL** | http://localhost:3000/customer/login |

### 👔 Employee Account (For Verifying Payments)

| Field | Value |
|-------|-------|
| **Employee ID** | `EMP001` |
| **Password** | `EmpSecurePass123!` |
| **Login URL** | http://localhost:3000/employee/login |

> **Note:** Employee registration is permanently disabled for security. Only pre-seeded accounts can access employee functions.

These accounts are created locally by `npm run prisma:seed`. They only exist in your own database.

---

## Manual test walkthrough

### ✅ Test 1: Customer Registration (Optional)

**If you want to create your own customer account:**

1. Navigate to: http://localhost:3000/customer/register
2. Fill in the registration form:
   - **Full Name**: Your Name (letters, spaces, hyphens only)
   - **ID Number**: 13 digits (e.g., `9901015800081`)
   - **Account Number**: 8-12 digits (e.g., `1234567890`)
   - **Username**: 3-30 characters (e.g., `johndoe123`)
   - **Email**: Valid email (e.g., `john@example.com`)
   - **Password**: Must include:
     - At least 8 characters
     - One uppercase letter
     - One lowercase letter
     - One number
     - One special character (e.g., `MyPass123!`)
3. Click **"Register"**
4. You should see: **"Registration successful! Redirecting to login..."**

**Expected Result:** ✅ Account created, redirected to login page

---

### ✅ Test 2: Customer Login and Payment Creation

#### Part A: Login

1. Navigate to: http://localhost:3000/customer/login
2. Enter credentials:
   - **Username**: `testcustomer`
   - **Account Number**: `12345678`
   - **Password**: `TestPass123!`
3. Click **"Login"**

**Expected Result:** ✅ Redirected to payments dashboard

#### Part B: Create a Valid Payment

1. Click **"Create New Payment"** button
2. Fill in the payment form with **VALID** data:
   - **Amount**: `250.50` (positive number, max 2 decimals)
   - **Currency**: `USD` (3 uppercase letters)
   - **Provider**: `SWIFT`
   - **Recipient Name**: `Jane Smith` (letters and spaces)
   - **Recipient Account**: `987654321012` (8-12 digits)
   - **SWIFT Code**: `ABCDUS33XXX` (8 or 11 characters, uppercase)
   - **Payment Reference**: `Invoice 12345` (optional, alphanumeric)
3. Click **"Submit Payment"**
4. Review details in confirmation modal
5. Click **"Confirm Payment"**

**Expected Result:** ✅ Success modal appears with:
- Transaction ID
- Payment details
- Status: PENDING
- Message: "Payment created successfully"

#### Part C: Test Validation Errors (IMPORTANT)

1. Click **"Create New Payment"** again
2. Enter **INVALID** data to test security:
   - **Amount**: `0` or `-50` (should reject)
   - **SWIFT Code**: `INVALID` or `ABC` (should reject)
   - **Recipient Account**: `123` (too short, should reject)
3. Click **"Submit Payment"**

**Expected Result:** ✅ Form shows validation errors:
- "Amount must be a positive number"
- "SWIFT code must be 8 or 11 characters"
- "Account number must be 8-12 digits"

#### Part D: View Payment History

1. Scroll down to **"Your Payments"** section
2. You should see your created payment(s) with:
   - Transaction ID
   - Amount and currency
   - Recipient details
   - Status (PENDING)
   - Creation date

**Expected Result:** ✅ All payments are listed with correct details

---

### ✅ Test 3: Employee Login and Payment Verification

#### Part A: Employee Login

1. **Logout** from customer account (click Logout button)
2. Navigate to: http://localhost:3000/employee/login
3. Enter credentials:
   - **Employee ID**: `EMP001`
   - **Password**: `EmpSecurePass123!`
4. Click **"Login"**

**Expected Result:** ✅ Redirected to employee dashboard

#### Part B: View Pending Payments

1. On the employee dashboard, you should see:
   - List of all payments in the system
   - Filter options (All, Pending, Verified, Submitted)
2. Click **"Pending"** filter to see only unverified payments

**Expected Result:** ✅ Payment created by testcustomer appears with status PENDING

#### Part C: Verify a Payment (Two-Factor Authentication)

1. Find the payment you created earlier
2. Click **"View Details"** to see full payment information
3. Click **"Verify"** button
4. **IMPORTANT:** The system will:
   - Request an action token (single-use security token)
   - Show a confirmation modal
5. Click **"Confirm Verification"**

**Expected Result:** ✅ 
- Payment status changes to **VERIFIED**
- Success message appears
- Action token is consumed and logged

#### Part D: Submit to SWIFT

1. Find the now-VERIFIED payment
2. Click **"Submit to SWIFT"** button
3. Another action token is requested (security measure)
4. Click **"Confirm Submission"**

**Expected Result:** ✅
- Payment status changes to **SUBMITTED**
- Success message appears
- In production, this would send to SWIFT network

---

### ✅ Test 4: Audit Logs (Security Monitoring)

1. While logged in as employee, click **"View Audit Logs"** in the header
2. You should see a comprehensive log of all system activities:
   - Customer registrations
   - Login attempts (successful and failed)
   - Payment creations
   - Payment verifications
   - Token requests and consumption
   - SWIFT submissions
3. Click **"Details"** on any log entry to see:
   - Timestamp
   - Entity type (Customer, Payment, Employee)
   - Action performed
   - IP address
   - User agent (browser info)
   - Metadata (additional details)

**Expected Result:** ✅ All actions are logged with tamper-evident hashing

---

### ✅ Test 5: Security Features Testing

#### Test 5A: Rate Limiting (Brute Force Protection)

**Note:** Only works if `DISABLE_RATE_LIMIT=false` in `.env`

1. Navigate to: http://localhost:3000/customer/login
2. Enter **wrong password** 3 times rapidly
3. On the 4th attempt, you should be blocked

**Expected Result:** ✅ "Too Many Requests" error after 3 failed attempts

#### Test 5B: XSS Protection (Cross-Site Scripting)

1. Try to register/login with username: `<script>alert('XSS')</script>`
2. The system should reject this input

**Expected Result:** ✅ Validation error: "Invalid characters in username"

#### Test 5C: Session Security

1. Login as customer
2. Open browser DevTools (F12)
3. Go to **Application** tab → **Cookies**
4. Find the `session` cookie
5. Note that it's marked as:
   - **HttpOnly**: ✅ (JavaScript cannot access it)
   - **Secure**: ✅ (HTTPS only in production)
   - **SameSite**: ✅ (CSRF protection)

**Expected Result:** ✅ Session cookie is secure

---

## Checklist

Use this checklist to verify all functionality:

- [ ] **Installation**: `npm install` completes successfully
- [ ] **Database Setup**: `npm run prisma:seed` creates test accounts
- [ ] **Server Start**: `npm run dev` starts without errors
- [ ] **Customer Registration**: New account can be created with valid data
- [ ] **Customer Registration Validation**: Invalid data is rejected with clear errors
- [ ] **Customer Login**: testcustomer can login successfully
- [ ] **Payment Creation**: Valid payment is created with PENDING status
- [ ] **Payment Validation**: Invalid payment data is rejected
- [ ] **Employee Login**: EMP001 can login successfully
- [ ] **Payment Verification**: Employee can verify PENDING payment
- [ ] **Action Token System**: Two-factor confirmation works for verify/submit
- [ ] **SWIFT Submission**: Verified payment can be submitted
- [ ] **Audit Logs**: All actions are logged and viewable
- [ ] **Audit Log Details**: Clicking details shows full information
- [ ] **Rate Limiting**: (If enabled) Blocks after 3 failed login attempts
- [ ] **Session Security**: Logout clears session and redirects
- [ ] **Employee Registration Block**: /api/employee/register returns 403

---

## Troubleshooting

### Problem: "Cannot connect to database"
**Solution:** 
1. Check your internet connection
2. Verify `DATABASE_URL` in `.env` file
3. Ensure MongoDB Atlas IP whitelist includes your IP

### Problem: "Too Many Requests" error
**Solution:** 
1. Set `DISABLE_RATE_LIMIT=true` in `.env`
2. Restart the server: `npm run dev`

### Problem: "Port 3000 already in use"
**Solution:**
1. Stop any other applications using port 3000
2. Or change the port: `npm run dev -- -p 3001`

### Problem: Test accounts don't work
**Solution:**
1. Re-run the seed script: `npm run prisma:seed`
2. Check for "already exists" message (accounts are already created)

### Problem: Page not loading
**Solution:**
1. Ensure server is running (check terminal for "Ready on http://localhost:3000")
2. Clear browser cache (Ctrl+Shift+Delete)
3. Try incognito/private browsing mode

