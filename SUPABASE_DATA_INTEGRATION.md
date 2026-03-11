# Supabase Data Integration Documentation

## Overview
All 3 admin panel pages have been successfully wired with real Supabase data. The pages now fetch live data from the `players`, `wagers`, and `wagers` (filtered for disputes) tables instead of using hardcoded placeholder data.

---

## Files Modified

### 1. `src/app/itszaadminlogin/users/page.tsx`
**Changes:**
- Added real-time Supabase query from `players` table with full schema fields
- Implemented `useEffect` + `useState` pattern for data fetching
- Added loading state with spinner (Loader component)
- Added error handling and display
- Real wallet addresses truncated to first 8 + last 4 characters
- Fallback to truncated wallet if username is null
- Status derived from `is_banned` field (true = "Banned", false = "Active")
- Ban/Unban buttons with confirmation dialog
- Calls `/api/admin/action` with `banPlayer` and `unbanPlayer` actions
- Earnings displayed in SOL format (divide lamports by 1,000,000,000)
- Maintained all existing styling, animations, and layout

**Key Features:**
- Search by wallet address or username
- Filter by all/active/banned status
- Ban players with reason (prompts in modal dialog)
- Unban players directly
- Optimistic UI updates after actions

### 2. `src/app/itszaadminlogin/wagers/page.tsx`
**Changes:**
- Added real-time Supabase query from `wagers` table
- Implemented expandable row detail view (no separate page)
- Displays full wager ID, match ID, player wallets with copy buttons
- Status mapping: `voting` → "In Progress", `disputed` → "Disputed", `resolved` → "Resolved", `cancelled` → "Cancelled", `created`/`joined` → "Pending"
- Action buttons only show for `voting`, `joined`, or `disputed` statuses
- Force Resolve: dropdown selector for player A or B as winner
- Force Refund: confirm dialog for refunding both players
- Transaction signature displayed with link to Solana explorer
- Calls `/api/admin/action` with `forceResolve` and `forceRefund` actions
- Maintained all styling and animations

**Key Features:**
- Search by wager ID or player wallets
- Filter by status
- Inline expanded details
- Copy wallet address buttons
- Dropdown to select winner for force resolve
- Transaction explorer link on success
- Optimistic UI updates (removes wager from list after action)

### 3. `src/app/itszaadminlogin/disputes/page.tsx`
**Changes:**
- Added real-time Supabase query from `wagers` table with `status = 'disputed'` filter
- Ordered by `created_at` ascending (oldest/most urgent first)
- Shows vote_player_a and vote_player_b for each dispute
- Time in dispute calculated from created_at (e.g., "6h ago", "1d ago")
- Force Resolve: dropdown to select winner + "type CONFIRM" confirmation input
- Force Refund: confirm dialog
- Transaction signature with explorer link
- Optimistic updates (removes dispute from list after resolution)
- Calls `/api/admin/action` with `forceResolve` and `forceRefund` actions
- Empty state: "No active disputes 🎉"
- Maintained all styling and animations

**Key Features:**
- Search by dispute ID, wager ID, or player wallets
- Expandable dispute details
- Vote display
- Force resolve with typed confirmation
- Force refund option
- Transaction confirmation display
- Auto-removes resolved disputes from list

---

## Database Schema Used

### players table
- `wallet_address` (TEXT, primary identifier)
- `username` (TEXT, nullable)
- `is_banned` (BOOLEAN)
- `ban_reason` (TEXT, nullable)
- `total_wins` (INTEGER)
- `total_losses` (INTEGER)
- `total_earnings` (BIGINT, in lamports)
- `created_at` (TIMESTAMPTZ)
- `flagged_for_review` (BOOLEAN)

### wagers table
- `id` (UUID)
- `match_id` (BIGINT)
- `player_a_wallet` (TEXT)
- `player_b_wallet` (TEXT)
- `game` (ENUM: chess | codm | pubg)
- `stake_lamports` (BIGINT, divide by 1,000,000,000 for SOL)
- `status` (ENUM: created | joined | voting | disputed | resolved | cancelled)
- `winner_wallet` (TEXT, nullable)
- `vote_player_a` (TEXT, nullable)
- `vote_player_b` (TEXT, nullable)
- `created_at` (TIMESTAMPTZ)
- `resolved_at` (TIMESTAMPTZ, nullable)

---

## Admin Action Endpoint

All admin actions route through `/api/admin/action` which calls the Supabase edge function `admin-action` with the following payloads:

### Ban Player
```json
{
  "action": "banPlayer",
  "adminWallet": "wallet_address",
  "walletAddress": "player_wallet",
  "reason": "ban reason"
}
```

### Unban Player
```json
{
  "action": "unbanPlayer",
  "adminWallet": "wallet_address",
  "walletAddress": "player_wallet"
}
```

### Force Resolve
```json
{
  "action": "forceResolve",
  "adminWallet": "wallet_address",
  "wagerId": "wager_uuid",
  "winnerWallet": "player_wallet"
}
```

### Force Refund
```json
{
  "action": "forceRefund",
  "adminWallet": "wallet_address",
  "wagerId": "wager_uuid"
}
```

---

## Features & UX

### Loading States
- Spinner displays while fetching initial data
- Action buttons show loading state (disabled + text change)
- Optimistic UI updates after successful actions

### Error Handling
- Fetch errors displayed in alert box
- Action errors displayed with error message
- Retry by manually fetching (users page) or submitting action again

### Data Formatting
- Wallet addresses truncated: first 8 + last 4 chars (e.g., `3h7fWbXX...j9dY6rQ5xN7k`)
- Lamports converted to SOL: `lamports / 1_000_000_000` with 4 decimal places
- Timestamps formatted as local date (e.g., `3/11/2026`) or relative time (e.g., `6h ago`)
- Games displayed capitalized (chess, codm, pubg)

### Search & Filter
- Live search filters by multiple fields (wallet, username, game, player names)
- Filter dropdowns for status categories
- Case-insensitive search

### Styling & Animations
- All components use existing glass morphism design
- Framer Motion animations preserved
- Dark theme with neon primary color
- Responsive layout (desktop and mobile)
- Consistent spacing and typography with gaming aesthetic

---

## Security Notes

- Wallet addresses verified through admin-action edge function
- Admin wallet required for all actions (from useWallet hook)
- Transaction signatures logged and displayed for audit trail
- RLS policies should be enabled on Supabase tables to restrict access
- Service role key required only on server-side (edge functions)

---

## Testing Checklist

- [ ] Users page loads with real player data
- [ ] Users can ban/unban players
- [ ] Wagers page loads with real wager data
- [ ] Wagers expandable view shows full details
- [ ] Force resolve works with winner selection
- [ ] Force refund works with confirmation
- [ ] Disputes page shows only disputed wagers
- [ ] Disputes can be resolved or refunded
- [ ] Transaction signatures display correctly
- [ ] Search and filter work on all pages
- [ ] Loading spinners show during fetches
- [ ] Error messages display on failures
- [ ] Optimistic updates work correctly
- [ ] Mobile responsive layout works
