# Images, layers, base images and tags

**Covers:** image, layer, base image, tag, latest tag, IDs

## The blueprint behind every container {#overview}
A container is something that runs. An image is what it runs from. If you mix the two up, many other things in Docker will not make sense. This article explains what is inside an image, why builds are usually fast, how to pick a starting point and how names and version labels work, including one that surprises almost everyone.

## image {#image}
An image is a read-only template that bundles an application with everything it needs to run: the files, the libraries and the settings. Starting an image creates a container, and doing so never changes the image. You can start the same image a hundred times and get a hundred separate containers.

A useful comparison is a class and its objects, or a recipe and the meals cooked from it. The image is the recipe. Each container is one cooked meal, and eating one does not alter the recipe.

## images {#images}
In the plural, images are the templates stored on your machine or in a registry. `docker images` lists the ones you have locally, with their sizes and how old they are.

## layer {#layer}
A layer is one step of an image, stacked on top of the previous ones. Each instruction in a Dockerfile that changes the file system, such as copying files or installing a package, adds a layer containing just those changes. The image is the stack of all its layers.

Docker caches each layer. When you rebuild, it reuses every layer whose instruction and inputs are unchanged, and rebuilds only from the first changed one onward. This is why a second build often takes seconds instead of minutes. Layers are also shared: ten images built on the same base image store the base once.

## layers {#layers}
"Layers", in the plural, refers to the stack as a whole. Two practical rules follow from how they work. Order the Dockerfile so that things that change rarely come first and things that change often come last. And remember that layers only add up: deleting a file in a later step does not shrink the image, because the earlier layer still contains it.

## base image {#base-image}
A base image is the image that your Dockerfile builds on top of, named in its first line. Choosing it is one of your most important decisions. A full operating system image is convenient but large. A slim variant, such as `python:3.12-slim`, has fewer files. An Alpine-based image is smaller still. Smaller bases download faster and contain fewer things that could have security flaws.

## tag {#tag}
A tag is a label attached to an image that names a specific version, such as `myapp:1.2`. The full name has the form `name:tag`. Tags let you choose exactly which build to run. If you leave the tag off, Docker uses a tag called `latest`.

A tag is only a pointer. It can be moved, so `myapp:1.2` could, after a rebuild, point at different content. For exact reproducibility, you can also refer to an image by its digest, a long fingerprint of its content.

## tags {#tags}
The plural just means the set of labels a repository has. A good habit is to give each build several tags: a precise version such as `1.4.2`, a shorter one such as `1.4` and a tag naming the code commit. Then you can pin precisely or follow along, as you prefer.

## latest tag {#latest-tag}
The `latest` tag is an ordinary tag, which many images use by default. It does not mean the newest build. It means whatever was last pushed without another tag, or whatever the publisher chose to give that name. It may be old, or it may be a different major version from the one you tested.

Using `latest` in production is risky, because the same command can produce different results on different days. Pin an explicit version instead.

## IDs {#ids}
IDs are the unique identifiers that Docker gives to things such as images and containers: a long string of letters and digits, usually shown shortened to the first 12 characters, like `3f9a1c7b2d4e`. You can use a unique start of an ID in commands, for example `docker stop 3f9a`.

## A worked example {#example}
A Dockerfile has these steps, each adding a layer:

| Order | Instruction | Layer size | Changes how often |
|---|---|---|---|
| 1 | `FROM python:3.12-slim` | 120 MB | Rarely |
| 2 | `COPY requirements.txt .` | 1 KB | Sometimes |
| 3 | `RUN pip install -r requirements.txt` | 80 MB | Sometimes |
| 4 | `COPY . .` | 2 MB | Every commit |

Rebuild after you edit one source file. Only layer 4 has changed, so layers 1 to 3 come from the cache. The rebuild takes about 2 seconds. If the order were reversed, with `COPY . .` before the install, then every code change would also invalidate the install layer, and each rebuild would download 80 MB of packages and take a minute or more.

The total size is 120 + 0.001 + 80 + 2, about 202 MB. Tagging the result `shop:1.4` and also `shop:1` gives two labels for one image, which share the same layers and the same ID, so there is no extra disk use.

## A common mistake {#mistake}
Using `latest` for anything you need to reproduce. A teammate who pulls it next month gets a different image from yours. Another mistake is putting a frequently changing step early in the Dockerfile, which destroys the cache.

## Key points {#key-points}
- An image is a read-only template. A container is a running instance of it.
- An image is a stack of cached layers. Put rarely changing steps first.
- A base image is the starting point, so choose a small, well-maintained one.
- Tags name versions, and `latest` is an ordinary tag that does not mean newest. Pin real versions.
