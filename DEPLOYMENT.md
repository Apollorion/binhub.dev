# Deployment Setup

This document explains how to set up automatic deployment to Cloudflare R2.

## 🔧 Required GitHub Secrets

You need to configure the following secrets in your GitHub repository settings:

### Required Secrets

1. **R2_ACCESS_KEY_ID**
   - Your Cloudflare R2 API Token access key ID
   - Get this from Cloudflare Dashboard → R2 → Manage R2 API tokens

2. **R2_SECRET_ACCESS_KEY**
   - Your Cloudflare R2 API Token secret access key
   - Get this from the same location as above

3. **R2_ACCOUNT_ID**
   - Your Cloudflare Account ID
   - Found in Cloudflare Dashboard → Right sidebar

4. **R2_BUCKET_NAME**
   - The name of your R2 bucket (e.g., `binhub-binaries`)

### Optional Secrets (for cache invalidation)

5. **CLOUDFLARE_ZONE_ID** _(optional)_
   - Your domain's zone ID if using a custom domain
   - Found in Cloudflare Dashboard → Your domain → Overview

6. **CLOUDFLARE_API_TOKEN** _(optional)_
   - API token with cache purge permissions
   - Create at Cloudflare Dashboard → My Profile → API Tokens

## 🚀 Setup Instructions

### Step 1: Create Cloudflare R2 Bucket

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Navigate to **R2 Object Storage**
3. Click **Create bucket**
4. Choose a name (e.g., `binhub-binaries`)
5. Select a location close to your users

### Step 2: Create R2 API Token

1. In Cloudflare Dashboard, go to **R2 → Manage R2 API tokens**
2. Click **Create API token**
3. Choose **Custom token**
4. Set permissions:
   - **Account**: `Your Account:read`
   - **Zone Resources**: `Include All zones`
   - **R2**: `Edit` (for your bucket)
5. Save the **Access Key ID** and **Secret Access Key**

### Step 3: Configure GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click **New repository secret**
4. Add each secret from the list above

### Step 4: Configure R2 Bucket for Public Access (Optional)

To make binaries publicly accessible:

1. Go to your R2 bucket settings
2. Enable **Public URL access** 
3. Or set up a custom domain for better URLs

## 🔄 Workflow Behavior

The GitHub Action will:

1. **Trigger on**: 
   - Push to `main` branch
   - Pull requests to `main` (for testing)

2. **Process**:
   - Install Python dependencies
   - Run `python processor.py` to download and organize binaries
   - Upload entire `output/` directory to R2
   - Set appropriate cache headers

3. **Upload Strategy**:
   - Uses `aws s3 sync` with `--delete` flag
   - Removes files from R2 that aren't in local output
   - Sets `Cache-Control: public, max-age=3600` headers

## 📝 Example Bucket Structure

After deployment, your R2 bucket will contain:

```
bucket-name/
├── api.json                     # Root API
├── index.html                   # Static site
├── j/
│   ├── api.json                # Letter API
│   └── jq/
│       ├── api.json            # Binary API
│       └── 1.6/
│           ├── api.json        # Version API
│           ├── linux-amd64/
│           │   └── jq          # Binary
│           └── darwin-amd64/
│               └── jq          # Binary
└── g/
    └── gh/
        └── 2.40.1/
            └── linux-amd64/
                └── gh
```

## 🌐 Custom Domain (Optional)

For prettier URLs like `https://binhub.dev/j/jq/1.6/linux-amd64/jq`:

1. Set up a custom domain in R2 bucket settings
2. Configure DNS to point to your R2 bucket
3. Add `CLOUDFLARE_ZONE_ID` and `CLOUDFLARE_API_TOKEN` secrets for cache invalidation

## 🐛 Troubleshooting

### Common Issues:

1. **403 Forbidden**: Check your API token permissions
2. **Bucket not found**: Verify `R2_BUCKET_NAME` and `R2_ACCOUNT_ID`
3. **Binaries not downloading**: Check network access and URLs in YAML files
4. **Large files failing**: R2 has a 5GB single-file limit (use multipart for larger)

### Debug Steps:

1. Check GitHub Actions logs
2. Verify all secrets are set correctly
3. Test R2 access with AWS CLI locally:
   ```bash
   aws s3 ls s3://your-bucket-name/ --endpoint-url https://YOUR_ACCOUNT_ID.r2.cloudflarestorage.com
   ```