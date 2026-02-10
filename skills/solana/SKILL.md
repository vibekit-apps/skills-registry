## ⚠️ MANDATORY: COPY THIS CODE EXACTLY - DO NOT MODIFY

### Step 1: Provider (copy exactly)
\`\`\`tsx
// app/providers.tsx
'use client';
import { SolanaProvider } from '@solana/react-hooks';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <SolanaProvider 
      cluster="mainnet-beta"
      endpoint="https://api.mainnet-beta.solana.com"
    >
      {children}
    </SolanaProvider>
  );
}
\`\`\`

### Step 2: Wallet Button (copy exactly)
\`\`\`tsx
// components/wallet-button.tsx
'use client';
import { useState } from 'react';
import { useWallet, useWalletSession, useWalletActions } from '@solana/react-hooks';

export function WalletButton() {
  const wallet = useWallet();
  const session = useWalletSession();
  const actions = useWalletActions();
  const [showModal, setShowModal] = useState(false);

  // Connected state
  if (wallet.status === 'connected' && session) {
    const address = session.account.address.toString();
    const shortAddress = address.slice(0, 4) + '...' + address.slice(-4);
    return (
      <div className="flex items-center gap-2">
        <span>{shortAddress}</span>
        <button onClick={() => actions.disconnectWallet()}>Disconnect</button>
      </div>
    );
  }

  // Connecting state
  if (wallet.status === 'connecting') {
    return <button disabled>Connecting...</button>;
  }

  // Disconnected state
  return (
    <>
      <button onClick={() => setShowModal(true)}>Connect Wallet</button>
      {showModal && (
        <div className="fixed inset-0 bg-black/50 flex items-center justify-center">
          <div className="bg-white p-6 rounded-lg">
            <h3 className="text-lg font-bold mb-4">Select Wallet</h3>
            <div className="space-y-2">
              <button 
                className="w-full p-2 border rounded"
                onClick={() => { actions.connectWallet('wallet-standard:phantom'); setShowModal(false); }}
              >
                Phantom
              </button>
              <button 
                className="w-full p-2 border rounded"
                onClick={() => { actions.connectWallet('wallet-standard:solflare'); setShowModal(false); }}
              >
                Solflare
              </button>
            </div>
            <button className="mt-4" onClick={() => setShowModal(false)}>Cancel</button>
          </div>
        </div>
      )}
    </>
  );
}
\`\`\`

### Step 3: Package.json dependencies (exact versions)
\`\`\`json
{
  "@solana/react-hooks": "^1.4.1",
  "@solana/client": "^0.7.0",
  "next": "14.2.15",
  "react": "^18",
  "react-dom": "^18"
}
\`\`\`

### CORRECT API (from @solana/react-hooks):
- \`useWallet()\` → returns { status: 'connected'|'connecting'|'disconnected'|'error' }
- \`useWalletSession()\` → returns session with session.account.address or undefined
- \`useWalletActions()\` → returns { connectWallet(id), disconnectWallet() }
- Call \`actions.connectWallet('wallet-standard:phantom')\` to connect
- Address: \`session.account.address.toString()\`

### ❌ DO NOT USE (these don't exist):
- useConnect, useDisconnect, useWalletAccount, useWallets
- wallet.address, wallet.name on session object
- createSolanaRpc with transport parameter
