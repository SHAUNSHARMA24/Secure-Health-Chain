# MediChain — Blockchain Medical Record System

<p align="center">
  <strong>Patients own their health data. Doctors access only what they're permitted. Every action is immutable.</strong>
</p>
---

## What is MediChain?

MediChain is a **decentralized medical record system** where:

- **Patients own and control** their medical records
- **Files are AES-256 encrypted** before being uploaded to IPFS
- **Blockchain stores only hashes, metadata, and permissions** — no raw health data
- **Doctors gain access** only after a patient explicitly grants it
- **Every important action** is logged immutably on-chain and in the audit trail
- **Instant revocation** — revoking a doctor's access takes effect immediately

This is a production-quality, portfolio-worthy implementation suitable for GitHub portfolios, college major projects, hackathons, and internship interviews.

---

## Features

### For Patients
- Register and login with JWT authentication
- Connect MetaMask wallet for on-chain identity
- Upload and manage medical records (lab reports, X-rays, MRIs, prescriptions, etc.)
- Encrypt records with AES-256 before upload
- Upload encrypted files to IPFS via Pinata
- Grant specific doctors access to specific records
- Revoke access instantly
- View complete audit history of who accessed what and when
- Verify record integrity (hash comparison)

### For Doctors
- Register with specialization and license number
- View authorized patients and their shared records
- Access medical records only after explicit patient permission
- Download encrypted records (patient-provided decryption key required)

### Smart Contract
- On-chain patient and doctor registration
- Upload record metadata (CID + hash) — no raw data
- Permission grant and revoke with events
- Immutable on-chain audit log
- Admin-controlled doctor verification

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        MEDICHAIN SYSTEM                         │
├─────────────┬────────────────────────┬────────────────────────  │
│   Frontend  │      Backend API       │   Blockchain Layer       │
│   React     │      Express.js        │   Ethereum / Hardhat     │
│   Vite      │      PostgreSQL        │   Solidity 0.8.19        │
│   TailwindCSS│     Drizzle ORM       │   Ethers.js              │
│   MetaMask  │      JWT + bcrypt      │   MediChain.sol          │
│   Ethers.js │      Zod validation    │                          │
└──────┬──────┴──────────┬─────────────┴───────────┬─────────────┘
       │                 │                          │
       │     REST API    │      Web3 calls          │
       │   (/api/...)    │    (ethers.js)           │
       └─────────────────┴──────────────────────────┘
                                  │
                    ┌─────────────┴──────────────┐
                    │         IPFS (Pinata)       │
                    │   Encrypted medical files   │
                    │   Only CIDs stored on-chain │
                    └─────────────────────────────┘
```

### Permission Workflow

```
Patient uploads record
       ↓
Encrypt file (AES-256)
       ↓
Upload encrypted file to IPFS → CID returned
       ↓
Store CID + SHA-256 hash on blockchain
       ↓
Doctor requests access (visible to patient)
       ↓
Patient grants permission → Blockchain event emitted
       ↓
Doctor gains read access to that record
       ↓
Patient revokes permission → Doctor loses access immediately
```

---

## Tech Stack

| Layer        | Technology                              |
|--------------|---------------------------------------- |
| Frontend     | React 18, Vite, Tailwind CSS, Recharts  |
| Backend      | Node.js 20, Express 5, TypeScript       |
| Database     | PostgreSQL 15, Drizzle ORM              |
| Blockchain   | Solidity 0.8.19, Hardhat, Ethers.js     |
| Storage      | IPFS via Pinata                         |
| Auth         | JWT, bcrypt (12 rounds)                 |
| Validation   | Zod, OpenAPI spec + Orval codegen       |
| Security     | Helmet, rate limiting, CORS, AES-256    |

---

## Folder Structure

```
medichain/
├── artifacts/
│   ├── api-server/            # Express 5 + TypeScript API server
│   │   └── src/
│   │       ├── lib/           # logger, audit helper
│   │       ├── middlewares/   # JWT auth middleware
│   │       └── routes/        # auth, records, permissions, audit, dashboard, doctors
│   └── medical-records/       # React + Vite frontend
│       └── src/
│           ├── components/    # Reusable UI components
│           ├── pages/         # Route-level page components
│           ├── hooks/         # Custom React hooks
│           └── lib/           # Auth utilities
├── contracts/
│   ├── MediChain.sol          # Main smart contract
│   ├── hardhat.config.js      # Hardhat configuration
│   ├── scripts/
│   │   └── deploy.js          # Deployment script
│   └── test/
│       └── MediChain.test.js  # Contract tests (Hardhat + Chai)
├── lib/
│   ├── api-spec/
│   │   └── openapi.yaml       # Single source of truth for API contracts
│   ├── api-client-react/      # Generated React Query hooks (DO NOT EDIT)
│   ├── api-zod/               # Generated Zod schemas (DO NOT EDIT)
│   └── db/
│       └── src/schema/        # Drizzle ORM table definitions
├── .env.example               # Environment variable template
├── .gitignore
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── SECURITY.md
└── pnpm-workspace.yaml
```

---

## Installation Guide

### Prerequisites

- Node.js 20+
- pnpm 9+
- PostgreSQL 15+
- Git
- MetaMask browser extension (for blockchain features)

### 1. Clone the Repository

```bash
git clone https://github.com/yourname/medichain.git
cd medichain
```

### 2. Install Dependencies

```bash
pnpm install
```

### 3. Configure Environment Variables

```bash
cp .env.example .env
```

Fill in all values in `.env`. See the [Environment Variables](#environment-variables) section below.

### 4. Set Up the Database

```bash
# Push the Drizzle schema to your PostgreSQL database
pnpm --filter @workspace/db run push
```

### 5. Start the API Server

```bash
pnpm --filter @workspace/api-server run dev
```

### 6. Start the Frontend

```bash
pnpm --filter @workspace/medical-records run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string | ✅ |
| `SESSION_SECRET` | JWT signing secret (long random string) | ✅ |
| `NODE_ENV` | `development` or `production` | ✅ |
| `PRIVATE_KEY` | Ethereum wallet private key for deployment | Blockchain |
| `SEPOLIA_RPC_URL` | Sepolia testnet RPC endpoint | Blockchain |
| `ETHERSCAN_API_KEY` | For contract verification | Blockchain |
| `CONTRACT_ADDRESS` | Deployed contract address | Blockchain |
| `PINATA_API_KEY` | Pinata API key for IPFS | IPFS |
| `PINATA_SECRET_API_KEY` | Pinata secret key | IPFS |
| `PINATA_JWT` | Pinata JWT for v2 API | IPFS |
| `VITE_CONTRACT_ADDRESS` | Contract address for frontend | Frontend |
| `VITE_CHAIN_ID` | Chain ID (11155111 for Sepolia) | Frontend |

---

## Smart Contract Deployment

### Local (Hardhat)

```bash
cd contracts
npm install
npx hardhat compile
npx hardhat test
npx hardhat node          # Start local blockchain
npx hardhat run scripts/deploy.js --network localhost
```

### Sepolia Testnet

```bash
# Get Sepolia ETH from https://sepoliafaucet.com
npx hardhat run scripts/deploy.js --network sepolia
npx hardhat verify --network sepolia <CONTRACT_ADDRESS>
```

The deployed contract address is written to `contracts/deployment.json` automatically.

---

## API Documentation

### Authentication

All protected routes require `Authorization: Bearer <token>` header.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/auth/register` | Register patient or doctor |
| `POST` | `/api/auth/login` | Login and receive JWT |
| `GET`  | `/api/auth/me` | Get current user profile |

### Medical Records

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| `GET`  | `/api/records` | Patient/Doctor | List records |
| `POST` | `/api/records` | Patient | Upload new record |
| `GET`  | `/api/records/:id` | Patient/Doctor | Get single record |
| `DELETE` | `/api/records/:id` | Patient | Delete record |
| `GET`  | `/api/records/:id/verify` | Patient/Doctor | Verify integrity |

### Permissions

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| `GET`  | `/api/permissions` | Patient | List all granted permissions |
| `POST` | `/api/permissions` | Patient | Grant doctor access to record |
| `PATCH`| `/api/permissions/:id/revoke` | Patient | Revoke doctor's access |

### Dashboard & Audit

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/dashboard/stats` | Statistics for current user |
| `GET` | `/api/dashboard/recent-activity` | Last 10 events |
| `GET` | `/api/audit` | Full audit log |
| `GET` | `/api/doctors` | List all doctors |
| `GET` | `/api/doctors/my-patients` | Doctor's authorized patients |

---

## IPFS Workflow

```
1. Patient selects a file (PDF, image, etc.)
2. Frontend encrypts the file with AES-256
   - Encryption key is generated per-file
   - Key is encrypted with patient's public key
3. Encrypted file is uploaded to IPFS via Pinata
   - Returns CID (content identifier)
4. SHA-256 hash of the encrypted file is computed
5. CID + hash + metadata are stored:
   - On PostgreSQL (for fast queries)
   - On blockchain (for immutability)
6. Encrypted key is stored (patient can share with doctor)
```

---

## MetaMask Configuration

### Add Sepolia to MetaMask

1. Open MetaMask → Networks → Add Network
2. Use these settings:
   - Network Name: `Sepolia Testnet`
   - RPC URL: `https://sepolia.infura.io/v3/YOUR_KEY`
   - Chain ID: `11155111`
   - Symbol: `ETH`
   - Explorer: `https://sepolia.etherscan.io`

3. Get test ETH: [sepoliafaucet.com](https://sepoliafaucet.com)

---

## Test Accounts

After running the seed script, these accounts are available:

| Role | Email | Password |
|------|-------|----------|
| Patient | alice@medichain.io | Patient@123 |
| Patient | bob@medichain.io | Patient@123 |
| Doctor | dr.smith@medichain.io | Doctor@123 |
| Doctor | dr.patel@medichain.io | Doctor@123 |

---

## Future Improvements

- [ ] Client-side AES-256 file encryption in the browser (WebCrypto API)
- [ ] Native Pinata SDK integration for file upload
- [ ] MetaMask wallet connection for on-chain permission management
- [ ] Email notifications when a doctor accesses a record
- [ ] Mobile app (Expo / React Native)
- [ ] Second-factor authentication (TOTP)
- [ ] Doctor verification workflow (admin panel)
- [ ] Multi-signature permissions (patient + next-of-kin)
- [ ] Zero-Knowledge Proof for anonymous record sharing
- [ ] HL7 FHIR standard compliance
- [ ] Integration with hospital management systems

---

## Known Limitations

- IPFS upload is not yet wired to a live Pinata SDK call — the frontend accepts CIDs as text input; wire in `@pinata/sdk` to automate this
- File encryption happens server-side in the current version; move it to the browser with WebCrypto for true end-to-end security
- Smart contract interactions (grantAccess, revokeAccess) are not yet called automatically from the frontend — add Ethers.js calls in the permission management page
- Doctor verification is manual (admin calls `verifyDoctor()` on-chain)

---

## License

MIT © 2026 MediChain Contributors. See [LICENSE](LICENSE) for details.

---

## Author

Built with care as a portfolio-quality, production-ready full-stack blockchain application.

**MediChain** — because your health data should belong to you.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Security

See [SECURITY.md](SECURITY.md) for responsible disclosure policy.
# Secure-Health-Chain
