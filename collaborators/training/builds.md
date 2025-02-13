
# Build Processes

## Why do we need builds?

As developers, we often prefer to create products in one format: writing content in Markdown, writing code in a static language (like TypeScript), writing templates, ...

But when we publish the product, we need the result in a different format: content in HTML, code in the execution language (like JavaScript), templates with the variable values inserted, ...

The build process transforms files from a source format into a target format. This transformation might involve compiling code, bundling assets, or generating static files.

## Common Build Scenarios

### Static Site Generation

The [NYJC Computing static site](https://github.com/nyjc-computing/nyjc-computing.github.io) is written in Markdown. A static site builder called [Jekyll](https://jekyllrb.com/) transform the files into the HTML that is required for display on [the actual site](https://nyjc-computing.github.io/).

## Workflow

The build step is typically required when:

1. Previewing the result
2. Deploying the product (i.e. sending the built code to the host server)
