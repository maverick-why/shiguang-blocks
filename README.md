# Wall of Blocks — Shiguang Brewing Co.

Static HTML + Supabase project for the Wall of Blocks marketing campaign.

## Setup

### 1. Configure Supabase

Open each HTML file and replace the placeholder constants at the top of the `<script>` section:

```js
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

For `admin.html`, also set:
```js
const SUPABASE_SERVICE_KEY = 'YOUR_SUPABASE_SERVICE_ROLE_KEY';
const ADMIN_PASSWORD = 'your-admin-password';
```

### 2. Run SQL in Supabase SQL Editor

```sql
-- Create the blocks table
CREATE TABLE blocks (
  id              uuid        DEFAULT gen_random_uuid() PRIMARY KEY,
  number          integer     UNIQUE NOT NULL CHECK (number BETWEEN 1 AND 100),
  name            text,
  mobile          text,
  message         text,
  public          boolean     DEFAULT true,
  status          text        DEFAULT 'available',
    -- available / reserved / engraving / installed
  payment_status  text        DEFAULT 'pending',
    -- pending / paid / refunded
  batch           text        DEFAULT 'Mar 30, 2026',
  install_photo   text,
  notes           text,
  created_at      timestamptz DEFAULT now(),
  paid_at         timestamptz
);

-- Seed 100 empty slots
INSERT INTO blocks (number)
SELECT generate_series(1, 100);

-- Enable Row Level Security
ALTER TABLE blocks ENABLE ROW LEVEL SECURITY;

-- Allow anyone to read all rows (name/message filtered in JS layer)
CREATE POLICY "public_read" ON blocks
  FOR SELECT USING (true);

-- Allow anon to claim an available block
CREATE POLICY "claim_block" ON blocks
  FOR UPDATE TO anon
  USING (status = 'available')
  WITH CHECK (
    status = 'reserved'
    AND payment_status = 'pending'
    AND name IS NOT NULL
    AND mobile IS NOT NULL
  );
```

### 3. Deploy

Deploy the folder as a static site:
- **GitHub Pages**: push to a repo, enable Pages on `main` branch root
- **Vercel**: `vercel --prod` from this directory

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | Landing page — wall grid + CTA |
| `claim.html` | 4-step claim flow |
| `success.html` | Digital certificate (Canvas) |
| `lookup.html` | Look up a claimed block |
| `mine.html` | My block status + progress |
| `admin.html` | Admin panel (password protected) |
| `styles.css` | Shared stylesheet |

## Colors

| Status | Color |
|--------|-------|
| Available | `#1e1e32` (dark) |
| Reserved (unpaid) | `#d4a017` (amber) |
| Paid | `#f0c040` (gold) |
| Installed | `#10b981` (green) |
