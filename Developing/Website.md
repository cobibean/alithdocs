# Developing the Alith Website

## Overview

This guide provides instructions for setting up a development environment for the Alith documentation website. The website is built using modern web technologies to showcase Alith's features, provide comprehensive documentation, and offer interactive examples. Whether you're contributing to the official Alith website or creating your own documentation site for your Alith-based project, this guide will help you get started.

> **🌐 Website Development**: The Alith website serves as both documentation and a showcase for the framework's capabilities, with interactive demos that highlight key features.

## Prerequisites

Before you begin developing the website, ensure you have the following:

* **Node.js**: Version 18.0.0 or later
* **npm**: Node package manager (comes with Node.js)
* **Git**: For version control
* **Optional**: Experience with React, Next.js, and Tailwind CSS

## Setup Instructions

### Installing Dependencies

First, clone the repository and install the dependencies:

```bash
# Clone the repository
git clone https://github.com/0xLazAI/alith.git
cd alith/website

# Install dependencies
npm install
```

### Running the Development Server

Start the development server to preview the website locally:

```bash
npm run dev
```

This will start the website on `http://localhost:3000`. The development server supports hot reloading, so any changes you make to the code will be immediately reflected in the browser.

## Project Structure

The website follows a standard Next.js structure with some additional directories:

```
website/
├── components/       # Reusable React components
├── pages/            # Next.js pages and routing
│   ├── docs/         # Documentation pages
│   ├── examples/     # Interactive examples
│   └── api/          # API endpoints
├── public/           # Static assets
├── styles/           # CSS and Tailwind configuration
├── lib/              # Utility functions and shared code
├── data/             # Content data (markdown, JSON)
└── scripts/          # Build and development scripts
```

## Adding New Content

### Adding Documentation Pages

Documentation pages are written in Markdown with MDX support for embedding React components:

1. Create a new Markdown file in the `pages/docs/` directory:

```bash
touch pages/docs/my-new-page.mdx
```

2. Add the front matter and content:

```mdx
---
title: My New Documentation Page
description: A comprehensive guide to a new feature
---

# My New Documentation Page

This is a new documentation page that explains a feature in detail.

## Section 1

Content for section 1...

## Section 2

Content for section 2...

<CodeExample language="typescript">
{`
const agent = new Agent({
  name: "MyAgent",
  model: "gpt-4",
  apiKey: process.env.OPENAI_API_KEY
});
`}
</CodeExample>
```

### Adding Interactive Examples

To add a new interactive example:

1. Create a new component in the `components/examples/` directory:

```tsx
// components/examples/MyNewExample.tsx
import { useState } from 'react';
import { Agent } from 'alith';

export default function MyNewExample() {
  const [input, setInput] = useState('');
  const [output, setOutput] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      // Example agent interaction
      const agent = new Agent({
        name: "ExampleAgent",
        model: "gpt-3.5-turbo",
        apiKey: process.env.NEXT_PUBLIC_OPENAI_API_KEY
      });
      
      const response = await agent.prompt(input);
      setOutput(response);
    } catch (error) {
      console.error('Error:', error);
      setOutput('An error occurred. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="example-container">
      <h2>Interactive Example</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Enter your prompt..."
          disabled={loading}
        />
        <button type="submit" disabled={loading}>
          {loading ? 'Processing...' : 'Submit'}
        </button>
      </form>
      {output && (
        <div className="output">
          <h3>Response:</h3>
          <p>{output}</p>
        </div>
      )}
    </div>
  );
}
```

2. Create a page that uses the example component:

```tsx
// pages/examples/my-new-example.tsx
import Layout from '../../components/Layout';
import MyNewExample from '../../components/examples/MyNewExample';

export default function MyNewExamplePage() {
  return (
    <Layout title="My New Example" description="Try out this new interactive example">
      <div className="container mx-auto px-4 py-8">
        <h1 className="text-3xl font-bold mb-6">My New Example</h1>
        <p className="mb-6">
          This example demonstrates a new feature of Alith. Try it out below!
        </p>
        <MyNewExample />
      </div>
    </Layout>
  );
}
```

## Building and Deploying

### Production Build

To create a production build of the website:

```bash
npm run build
```

This will generate optimized static files in the `.next` directory.

### Preview Production Build

To preview the production build locally:

```bash
npm run start
```

### Deployment

The website can be deployed to various platforms:

#### Vercel (Recommended)

```bash
npm install -g vercel
vercel
```

#### Netlify

```bash
npm install -g netlify-cli
netlify deploy
```

## Advanced Customization

### Theming

The website uses Tailwind CSS for styling. You can customize the theme by editing the `tailwind.config.js` file:

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          // ... other shades
          900: '#0c4a6e',
        },
        // Add your custom colors
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
        mono: ['Fira Code', 'monospace'],
      },
    },
  },
  // Other configurations
};
```

### Adding API Endpoints

Create serverless API endpoints in the `pages/api/` directory:

```tsx
// pages/api/example.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { Agent } from 'alith';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  const { prompt } = req.body;

  if (!prompt) {
    return res.status(400).json({ message: 'Prompt is required' });
  }

  try {
    const agent = new Agent({
      name: "APIAgent",
      model: "gpt-4",
      apiKey: process.env.OPENAI_API_KEY
    });

    const response = await agent.prompt(prompt);
    return res.status(200).json({ response });
  } catch (error) {
    console.error('API Error:', error);
    return res.status(500).json({ message: 'Internal server error' });
  }
}
```

## Best Practices

* **Mobile Responsiveness**: Ensure all pages and components are responsive
* **Accessibility**: Follow WCAG guidelines for accessible web content
* **Performance**: Optimize images and minimize JavaScript bundles
* **SEO**: Use proper meta tags and semantic HTML
* **Documentation**: Keep code examples up-to-date with the latest Alith API

> **💡 Tip:** When adding interactive examples, provide both TypeScript and Python code samples to accommodate users of different SDKs.

## Common Issues and Solutions

### Issue 1: Environment Variables

**Problem**: Environment variables not working in the browser.

**Solution**: In Next.js, client-side environment variables must be prefixed with `NEXT_PUBLIC_`:

```bash
# .env.local
NEXT_PUBLIC_OPENAI_API_KEY=your-api-key
```

### Issue 2: Build Errors

**Problem**: Build fails due to TypeScript errors.

**Solution**: Check for type issues and ensure all required dependencies are installed:

```bash
# Run TypeScript check
npm run type-check

# Check for missing dependencies
npm ls --missing
```

## Contributing Guidelines

If you're contributing to the Alith website:

1. **Fork the Repository**: Create your own fork of the repository
2. **Create a Feature Branch**: `git checkout -b feature/my-feature`
3. **Make Your Changes**: Implement your feature or bug fix
4. **Test Locally**: Ensure the website works correctly in development mode
5. **Build and Preview**: Verify the production build works as expected
6. **Submit a Pull Request**: With a detailed description of your changes

## Related Documentation

* [GetStarted.md](../GetStarted.md) - General getting started guide
* [Developing/NodeJs.md](NodeJs.md) - Node.js development environment setup
* [Integrations/Eliza.md](../Integrations/Eliza.md) - Integrate with ElizaOS
* [Features/Tools.md](../Features/Tools.md) - Learn about Alith tools

## Frequently Used APIs

```typescript
// Common Next.js page structure
import { GetStaticProps } from 'next';
import Layout from '../components/Layout';

interface PageProps {
  data: any;
}

export default function Page({ data }: PageProps) {
  return (
    <Layout title="Page Title" description="Page description">
      <div className="container">
        {/* Page content */}
      </div>
    </Layout>
  );
}

export const getStaticProps: GetStaticProps = async () => {
  // Fetch data at build time
  const data = /* fetch data */;
  
  return {
    props: {
      data,
    },
    // Revalidate the data every hour
    revalidate: 3600,
  };
};
```

---

*Last Updated: 2025-03-16*