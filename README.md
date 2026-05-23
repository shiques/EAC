# EAC AutoPass — Mobile App

Smart cross-border vehicle movement platform for East African Community member states.

## Tech Stack

- **Framework**: React Native + Expo SDK 51
- **Navigation**: React Navigation v6 (Stack + Bottom Tabs)
- **Icons**: @expo/vector-icons (MaterialCommunityIcons)
- **Payments**: M-Pesa Daraja API (STK Push), Visa/Mastercard, Mobile Money
- **Build**: Expo Application Services (EAS)

---

## Quick Start

```bash
# Install dependencies
npm install

# Start development server
npx expo start

# Run on Android emulator
npx expo start --android

# Run on iOS simulator
npx expo start --ios
```

---

## App Screens

| Screen            | Description                                     |
|-------------------|-------------------------------------------------|
| SplashScreen      | Animated logo + EAC flag colours                |
| OnboardingScreen  | 4-slide feature carousel                        |
| LoginScreen       | Phone/ID + password + biometric login           |
| RegisterScreen    | Full account creation                           |
| HomeScreen        | Dashboard with active permits + live traffic    |
| ApplyScreen       | 5-step permit application wizard                |
| PaymentScreen     | M-Pesa STK push + card + mobile money + bank    |
| PermitScreen      | Issued digital permit with QR code              |
| MyPermitsScreen   | All permits with status filters                 |
| ScannerScreen     | Border officer QR/ANPR/NFC/RFID scanner         |
| ProfileScreen     | User account, preferences, compliance score     |

---

## M-Pesa STK Push Integration

### How it works

1. User enters their Safaricom number on PaymentScreen
2. App sends a request to your backend server
3. Backend calls the **Safaricom Daraja API** (STK Push endpoint)
4. User receives a prompt on their phone — enters M-Pesa PIN
5. App polls your backend for confirmation
6. On success, permit is issued and PermitScreen is shown

### Backend endpoint (Node.js example)

```js
// POST /api/payments/mpesa/stkpush
const axios = require('axios');

async function getAccessToken() {
  const auth = Buffer.from(`${CONSUMER_KEY}:${CONSUMER_SECRET}`).toString('base64');
  const res  = await axios.get('https://sandbox.safaricom.co.ke/oauth/v1/generate?grant_type=client_credentials', {
    headers: { Authorization: `Basic ${auth}` },
  });
  return res.data.access_token;
}

app.post('/api/payments/mpesa/stkpush', async (req, res) => {
  const { phone, amount, accountRef, description } = req.body;
  const token     = await getAccessToken();
  const timestamp = new Date().toISOString().replace(/[-T:.Z]/g, '').slice(0, 14);
  const password  = Buffer.from(`${SHORTCODE}${PASSKEY}${timestamp}`).toString('base64');

  const response = await axios.post(
    'https://sandbox.safaricom.co.ke/mpesa/stkpush/v1/processrequest',
    {
      BusinessShortCode: SHORTCODE,
      Password:          password,
      Timestamp:         timestamp,
      TransactionType:  'CustomerPayBillOnline',
      Amount:            amount,
      PartyA:            phone,
      PartyB:            SHORTCODE,
      PhoneNumber:       phone,
      CallBackURL:       'https://api.eacautopass.com/mpesa/callback',
      AccountReference:  accountRef,
      TransactionDesc:   description,
    },
    { headers: { Authorization: `Bearer ${token}` } }
  );

  res.json(response.data);
});
```

### Daraja API credentials

1. Register at https://developer.safaricom.co.ke
2. Create an app → get Consumer Key and Consumer Secret
3. Use sandbox for testing, production for live payments
4. Set `.env`:

```env
MPESA_CONSUMER_KEY=your_consumer_key
MPESA_CONSUMER_SECRET=your_consumer_secret
MPESA_SHORTCODE=174379
MPESA_PASSKEY=your_passkey
MPESA_CALLBACK_URL=https://api.eacautopass.com/mpesa/callback
```

---

## Build for App Stores

### Prerequisites

```bash
npm install -g eas-cli
eas login
```

### Android (Google Play Store)

```bash
# Production AAB (for Play Store)
eas build --platform android --profile production

# Preview APK (for internal testing)
eas build --platform android --profile preview

# Submit to Play Store
eas submit --platform android --profile production
```

### iOS (Apple App Store)

```bash
# Production build
eas build --platform ios --profile production

# Submit to App Store Connect
eas submit --platform ios --profile production
```

---

## Environment Variables

Create `.env` in the project root:

```env
EXPO_PUBLIC_API_URL=https://api.eacautopass.com
EXPO_PUBLIC_MPESA_SHORTCODE=174379
EXPO_PUBLIC_GOOGLE_MAPS_KEY=your_key
```

---

## Project Structure

```
EACAutoPass/
├── App.js                        # Root component
├── app.json                      # Expo config (name, icons, permissions)
├── eas.json                      # EAS build profiles
├── babel.config.js               # Babel preset
├── package.json                  # Dependencies
└── src/
    ├── constants/
    │   └── theme.js              # Colours, spacing, permit types, borders
    ├── navigation/
    │   └── RootNavigator.js      # Stack + Tab navigators + custom tab bar
    └── screens/
        ├── SplashScreen.js
        ├── OnboardingScreen.js
        ├── LoginScreen.js
        ├── RegisterScreen.js
        ├── HomeScreen.js
        ├── ApplyScreen.js        # 5-step permit wizard
        ├── PaymentScreen.js      # M-Pesa STK push + card + bank
        ├── PermitScreen.js       # Issued permit + QR code
        ├── MyPermitsScreen.js    # Permit list with filters
        ├── ScannerScreen.js      # Border officer scanner
        └── ProfileScreen.js      # User account + compliance score
```

---

## Legal & Compliance

- EACCMA (East African Community Customs Management Act)
- COMESA Yellow Card insurance verification
- Kenya Data Protection Act 2019
- Safaricom M-Pesa Daraja API Terms
- Apple App Store Review Guidelines
- Google Play Developer Program Policies
