# Image Download Guide

I've updated the homepage to use three new images. Here's how to get free, copyright-free images:

## Required Images

Save these images to `/assets/images/` folder:

### 1. **tech-stack.png** (Technical Expertise Section)

**Recommended sources:**

- **Unsplash**: Search "technology workspace" or "coding setup"
  - Example: https://unsplash.com/s/photos/programming
  - Download any high-quality tech/coding image (1200x800px minimum)
- **Alternative - Create a tech collage:**
  - Visit https://worldvectorlogo.com/ or https://simpleicons.org/
  - Download SVG logos for: Kubernetes, MongoDB, React, Docker, AWS
  - Use a free tool like Canva to arrange them in a grid

**Quick command to download an Unsplash image:**

```bash
# Replace IMAGE_ID with actual image ID from Unsplash
curl -L "https://unsplash.com/photos/IMAGE_ID/download?force=true" -o assets/images/tech-stack.png
```

### 2. **professional.png** (Professional Experience Section)

**Recommended sources:**

- **Unsplash**: Search "team meeting" or "collaboration" or "office"
  - https://unsplash.com/s/photos/professional-work
  - Download image showing teamwork/professional environment

**Suggested searches:**

- "software development team"
- "agile team meeting"
- "tech collaboration"

### 3. **team-value.png** (What I Bring Section)

**Recommended sources:**

- **Unsplash**: Search "leadership" or "mentorship" or "success"
  - https://unsplash.com/s/photos/leadership
  - Download image showing growth/success/teamwork

**Suggested searches:**

- "business success"
- "team achievement"
- "professional growth"

## Easy Download Method

### Option 1: Use Unsplash (Recommended)

1. Go to https://unsplash.com/
2. Search for suggested terms above
3. Click on image you like
4. Click "Download free" button
5. Rename and move to `assets/images/` folder

### Option 2: Use Pexels

1. Go to https://www.pexels.com/
2. Search for same terms
3. Download and save to `assets/images/`

### Option 3: Quick Terminal Downloads

Here are some curated free images you can download directly:

```bash
cd assets/images/

# Technical Expertise - Programming workspace
curl -L "https://images.unsplash.com/photo-1461749280684-dccba630e2f6?w=1200" -o tech-stack.png

# Professional Experience - Team collaboration
curl -L "https://images.unsplash.com/photo-1522071820081-009f0129c71c?w=1200" -o professional.png

# Team Value - Success/Growth
curl -L "https://images.unsplash.com/photo-1552664730-d307ca884978?w=1200" -o team-value.png
```

## Image Requirements

- **Format**: PNG or JPG
- **Size**: At least 800x600px (1200x800px recommended)
- **Orientation**: Landscape (horizontal)
- **Quality**: High resolution for crisp display

## After Adding Images

Run the development server to see changes:

```bash
npm run dev
```

Visit http://localhost:1313 to view your updated homepage!

## Copyright Info

All images from Unsplash and Pexels are:

- ✅ Free to use
- ✅ No attribution required (though appreciated)
- ✅ Can be used commercially
- ✅ Can be modified

**License**: https://unsplash.com/license
