# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Cloudflare Workers-based AI image generation service that supports text-to-image, image-to-image, and inpainting capabilities. The application is built as a single-page web application using vanilla JavaScript with Cloudflare Workers AI models.

## Architecture

- **worker.js**: Main Cloudflare Worker entry point containing all backend logic
  - Handles API routes: `/api/models`, `/api/prompts`, `/api/config`, `/api/auth`
  - Processes image generation requests (POST `/`)
  - Manages authentication via passwords and cookies
  - Integrates with Cloudflare Workers AI binding
- **index.html**: Complete frontend SPA with embedded styles and JavaScript
  - Responsive UI with dark/light theme support
  - Image generation controls and parameter management
  - Gallery view for multiple image outputs
  - Progress tracking and result display
- **wrangler.toml**: Cloudflare Workers configuration
  - Defines Workers AI binding as `AI`
  - Sets environment variables including `ADMIN_PASSWORD`
  - Configures HTML text module imports

## Development Commands

### Deployment
```bash
wrangler deploy
```

### Local Development  
```bash
wrangler dev
```

### Configuration Management
```bash
# Set password via secret (recommended for production)
wrangler secret put ADMIN_PASSWORD

# List secrets
wrangler secret list

# Delete secret
wrangler secret delete ADMIN_PASSWORD
```

## Key Features

### Supported AI Models
- `stable-diffusion-xl-base-1.0`: Text-to-image, high resolution
- `flux-1-schnell`: Fast text-to-image with JSON response format
- `dreamshaper-8-lcm`: LCM optimized for artistic styles
- `stable-diffusion-xl-lightning`: Ultra-fast generation
- `stable-diffusion-v1-5-img2img`: Image-to-image transformation
- `stable-diffusion-v1-5-inpainting`: Masked region editing

### Authentication
- Password protection via `ADMIN_PASSWORD` or `ADMIN_PASSWORDS` env vars
- Cookie-based session management (`auth=1`)
- Login overlay for password-protected instances

### Image Generation Capabilities
- Batch generation (1-8 images)
- Progress tracking with estimated completion
- ZIP download for multiple images
- Parameter copying and metadata display
- Gallery view with hover actions (zoom, copy, download)

## Code Patterns

### Model Configuration
Models are defined in `AVAILABLE_MODELS` array with metadata:
```javascript
{
  id: 'model-id',
  name: 'Display Name', 
  description: 'Description',
  key: '@cf/provider/model-name',
  requiresImage: boolean,
  requiresMask: boolean
}
```

### Error Handling
- Comprehensive error responses with details
- Client-side validation for required fields
- Server-side image URL validation and size limits
- Timeout handling (60s) with AbortController

### Response Format
- Single image: Binary PNG with custom headers
- Multiple images: JSON with base64 dataURL array
- Headers: `X-Used-Model`, `X-Server-Seconds`, `X-Image-Bytes`

## Important Notes

- No package.json or build process - pure Cloudflare Workers
- HTML is imported as text module via wrangler rules
- All dependencies loaded via CDN (Tailwind, FontAwesome, JSZip)
- Image processing uses Cloudflare Workers AI binding (`env.AI.run`)
- FLUX model returns JSON format, others return binary
- 10MB image size limit for img2img/inpainting inputs