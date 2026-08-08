# osctrl docs

<p align="center">
  <img alt="osctrl" src="osctrl.png" width="300" />
  <p align="center">
    Fast and efficient osquery management.
  </p>
</p>

Documentation for [osctrl](https://github.com/jmpsec/osctrl): https://docs.osctrl.net

## Commands

```bash
npm install
npm run dev      # local dev server on :4321
npm run build    # static output into dist/
npm run preview  # serve the built output
```

## Layout

```
├── astro.config.mjs          # site config, explicit sidebar, fonts
├── deploy-example.yml        # GitHub Pages workflow (not active yet)
├── src/
│   ├── content.config.ts     # Starlight docs collection
│   ├── content/docs/         # the Markdown, one file per page
│   │   ├── index.mdx         # splash landing page
│   │   ├── components/       # 1. Components
│   │   ├── deployment/       # 2. Deployment
│   │   ├── configuration.md  # 3. Configuration
│   │   ├── usage/            # 4. Usage (incl. osctrl-cli subcommands)
│   │   └── contributing.md   # 5. Contributing
│   ├── assets/               # images referenced from front matter
│   └── styles/osctrl.css     # osctrl design tokens mapped onto Starlight
└── public/                   # copied verbatim into the build
    ├── img/                  # diagrams and gifs
    ├── fonts/                # self-hosted Bai Jamjuree wordmark
    └── openapi/doc.html      # Stoplight Elements API reference, unchanged
```
