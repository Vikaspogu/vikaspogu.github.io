# vikaspogu.dev

[![Deploy Hugo site to Pages](https://github.com/vikaspogu/vikaspogu.github.io/actions/workflows/pages.yaml/badge.svg)](https://github.com/vikaspogu/vikaspogu.github.io/actions/workflows/pages.yaml)

🚀 Personal blog and portfolio site focused on DevOps, Cloud Native technologies, and software development.

[🌐 Visit Site ↗](https://vikaspogu.dev/)

## About

This is my personal technical blog where I share insights, tutorials, and experiences with:

- **DevOps & CI/CD**: Jenkins, Tekton Pipelines, GitOps workflows
- **Cloud Native**: Kubernetes, OpenShift, ArgoCD, Helm
- **Infrastructure**: Monitoring, Security, Networking
- **Development**: Go, Java, React, Spring Boot
- **Home Lab**: Raspberry Pi projects, Self-hosting solutions

## Content Areas

- **Blog Posts**: Technical tutorials and deep-dives on various DevOps and cloud native topics
- **Documentation**: Reference materials and guides
- **Projects**: Personal and professional project showcases

## Deployment

This site is automatically deployed to GitHub Pages using GitHub Actions. The deployment workflow is configured in [`.github/workflows/pages.yaml`](./.github/workflows/pages.yaml).

The site is automatically built and deployed when changes are pushed to the `master` branch. You can also [run the workflow manually](https://docs.github.com/en/actions/using-workflows/manually-running-a-workflow) if needed.

## Local Development

Pre-requisites: [Hugo](https://gohugo.io/getting-started/installing/), [Go](https://golang.org/doc/install) and [Git](https://git-scm.com)

```shell
# Clone the repo
git clone https://github.com/vikaspogu/vikaspogu.github.io.git

# Change directory
cd vikaspogu.github.io

# Install dependencies
hugo mod tidy

# Start the development server
hugo server --logLevel debug --disableFastRender -p 1313
```

The site will be available at `http://localhost:1313`

### Update theme

```shell
hugo mod get -u
hugo mod tidy
```

See [Update modules](https://gohugo.io/hugo-modules/use-modules/#update-modules) for more details.

## Technology Stack

This site is built with:

- **[Hugo](https://gohugo.io/)**: Fast static site generator
- **[Hextra Theme](https://github.com/imfing/hextra)**: Modern documentation theme
- **GitHub Pages**: Free static site hosting
- **GitHub Actions**: Automated CI/CD pipeline

## Featured Topics

Some popular blog posts cover:

- Kubernetes cluster setup and management
- OpenShift development and deployment
- CI/CD pipelines with Jenkins and Tekton
- GitOps workflows with ArgoCD
- Infrastructure monitoring and observability
- Home lab projects with Raspberry Pi
- Container security and best practices

## License

This project is open source and available under the [MIT License](LICENSE).