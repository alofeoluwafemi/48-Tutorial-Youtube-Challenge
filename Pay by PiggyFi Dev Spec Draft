## Register

- Take the following fields
  - Firstname
  - Lastname
  - Email
  - Phone number
  - Country Iso2 code
  - invite code (referral code) - optional
  - Password
  - Agree to T&C
- Success response 200
- Validation error response 422
- User exist response 409

## Send Email/Phone Verification Code

- Incorrect Validation code response 422
- Expired Validation code response 410
- Verification success response 200

## Logout API

- Return response success (delete session or cookie)

## Add you business

- Take the following fields
  - Business name
  - Business email
  - Address Line 1
  - Address Line 2 (Not required)
  - City
  - Postcode
  - Country (Fixed on selected during registration)
  - Tax number (not required)
- Success response 200
- Validation error response 422
- Provide end point to update details

## Update Profile

- Take the following fields
  - Firstname
  - Lastname
  - Phone number

## Add withdrawal methods

- Take the following fields
  - Method (Wise, Bank, Mobile Money, Wallet) - only required field
  - Bank Name
  - Bank Account Number
  - Wise Email
  - Mobile money account number
  - Bank Name
  - Wallet Address
  - Network (CELO only for now)
- Success response 200
- Provide endpoint to delete withdrawal method

## Withdraw
- Send OTP to phone number added to profile
- Take the following fields
  - Withdrawal method ID (foreign key)
  - currency (cUSD, CELO, cEURO, cREAL)
  -

## Add Client

- Take the following fields
  - Person name
  - Company name
  - Email
- Success response 200
- Provide enpoint to delete client
- Provide enpoint to update client

## Dashboard

- Return the following as a single request
  - asset balances
  - recent transactions
  - revenue history for graph
  - account setup (boolean)
  - total balance

## Create Invoice

- Get next Invoice number to use
- Take the following fields
  - attachment
  - invoice number
  - note
  - items
    - description
    - quantity
    - unit price
    - amount
    - discount
    - shipping
    - tax
- Success response 200 for create invoice
- Success endpoin 200 for send invoice

## List Invoices

- All
- Draft
- Awaiting Payment
- Paid
- Overdue
- Rejected

- Also allow for pagination (50 rows per page)

## Get Invoice Timeline

## Download Invoice

## Download Receipt

## View on Celo Explorer

## Export as CSV
