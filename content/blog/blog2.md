---
title: "Streamlining Docker Workflows with GNU Make" 
date: 2025-03-01
lastmod: 2025-03-01
tags: ["GNU Make", "docker", "podman", "automation", "reproducibility"]
author: ["Blazej M. Baczkowski"]
description: ""
summary: "How to build, test, and run Docker containers automatically with GNU Make?"
editPost:
    URL: ""
    Text: ""
showToc: true
disableAnchoredHeadings: false

---


As data scientists, we often find ourselves juggling dependencies, environment setups, and reproducibility issues. One way to streamline this process is by using **Docker** (or **Podman**) to create containerized environments. However, repeatedly typing long `docker build` and `docker run` commands can be tedious. This is where magnificent [**GNU Make**](https://www.gnu.org/software/make/) comes in handy—it allows us to automate these commands efficiently. And is a recipe for others reproducing our projects. 

In this post, we’ll explore how to build, test, and run a **Docker** (or **Podman**) container using repeatable workflows with **GNU Make** 

We will explain everything working with the following `Makefile` snippet to build and run `Quarto` in a container, which we described in a previous [post](/website/blog/blog1). 

```make
.PHONY: all build test save load clean

# Detect whether podman or docker is available
ENGINE := $(shell command -v podman >/dev/null 2>&1 && echo podman || echo docker)

# Image name and tag
IMG_NAME := r-quarto
TAG := latest
IMG := $(IMG_NAME):$(TAG)

# Define build context
BUILD_CONTEXT := r-quarto
DOCKERFILE := $(BUILD_CONTEXT)/Dockerfile

# Ensure build dependencies exist
BUILD_DEPS := $(DOCKERFILE) $(BUILD_CONTEXT)/r-pkgs.txt $(BUILD_CONTEXT)/tex-pkgs.txt

# Default target (build and test)
all: build test

# Build the Docker image
build: $(BUILD_DEPS)
	$(ENGINE) build -t $(IMG) -f $(DOCKERFILE) $(BUILD_CONTEXT)

# Run a test container interactively
test:
	$(ENGINE) run --rm --user=root -it -v $(PWD):/home/dockeruser $(IMG) bash

# Save the image as a tarball for easy sharing
save:
	$(ENGINE) save $(IMG) | gzip > $(IMG_NAME).tar.gz
	@echo "Docker image saved as $(IMG_NAME).tar.gz"

# Load a previously saved image
load:
	gunzip -c $(IMG_NAME).tar.gz | $(ENGINE) load
	@echo "Docker image $(IMG) loaded"

# Remove the image (clean up)
clean:
	$(ENGINE) rmi -f $(IMG)
	@echo "Docker image $(IMG) removed"
```


---

## **Understanding the Makefile**  

Let's break down the key components of our `Makefile`.  

---

### **1. Declaring `.PHONY` Targets**
```make
.PHONY: all build test save load clean
```
- Declares **phony** targets, meaning these are not actual files but just labels for actions.
- Ensures that `make` always executes the commands for these targets, even if files with the same names exist.

---

### **2. Selecting the Container Engine**
```make
ENGINE := $(shell command -v podman >/dev/null 2>&1 && echo podman || echo docker)
```
- Checks whether `podman` is available.
- If `podman` exists, it sets `ENGINE=podman`; otherwise, it defaults to `docker`.
- Ensures the same commands work regardless of whether Podman or Docker is installed.

---

### **3. Defining Variables**
```make
IMG_NAME := r-quarto
TAG := latest
IMG := $(IMG_NAME):$(TAG)
```
- **`IMG_NAME`** is the base name of the container image (`r-quarto`).
- **`TAG`** is set to `latest`, meaning the most recent version.
- **`IMG`** combines the two: `r-quarto:latest`.

```make
BUILD_CONTEXT := r-quarto
DOCKERFILE := $(BUILD_CONTEXT)/Dockerfile
BUILD_DEPS := $(DOCKERFILE) $(BUILD_CONTEXT)/r-pkgs.txt $(BUILD_CONTEXT)/tex-pkgs.txt
```
- Defines the **build context** (where the Docker build files are located).
- **`DOCKERFILE`** points to `r-quarto/Dockerfile`.
- **`BUILD_DEPS`** lists files required for building the image:
  - `Dockerfile`
  - `r-pkgs.txt` (likely a list of R packages)
  - `tex-pkgs.txt` (likely a list of LaTeX packages)

---

### **4. Default Target (`all`)**
```make
all: build test
```
- Runs both `build` and `test` when you just type `make`.

---

### **5. Building the Image**
```make
build: $(BUILD_DEPS)
	$(ENGINE) build -t $(IMG) -f $(DOCKERFILE) $(BUILD_CONTEXT)
```
- Builds the container image using **Podman or Docker**.
- Ensures required files (`BUILD_DEPS`) exist before building.
- Tags the image as `r-quarto:latest`.

---

### **6. Running a Test Container**
```make
test:
	$(ENGINE) run --rm --user=root -it -v $(PWD):/home/dockeruser $(IMG) bash
```
- Starts a container **interactively** (`-it`) with a **root user**.
- Mounts the current directory (`$(PWD)`) inside the container at `/home/dockeruser`.
- Runs a `bash` shell.
- **`--rm`** removes the container after exit.

---

### **7. Saving the Image to a Compressed Tarball**
```make
save:
	$(ENGINE) save $(IMG) | gzip > $(IMG_NAME).tar.gz
	@echo "Docker image saved as $(IMG_NAME).tar.gz"
```
- Exports (`save`) the container image to a `.tar.gz` file for easy sharing.
- Uses `gzip` to compress it.
- Prints a confirmation message.

---

### **8. Loading a Saved Image**
```make
load:
	gunzip -c $(IMG_NAME).tar.gz | $(ENGINE) load
	@echo "Docker image $(IMG) loaded"
```
- **Uncompresses** (`gunzip -c`) the saved `.tar.gz` file.
- **Loads** the image into Docker/Podman.
- Prints a confirmation message.

---

### **9. Cleaning Up (Removing the Image)**
```make
clean:
	$(ENGINE) rmi -f $(IMG)
	@echo "Docker image $(IMG) removed"
```
- Removes (`rmi -f`) the image from the system.
- Prints a confirmation message.

---

## **How to Use This Makefile**
| Command | What it Does |
|---------|-------------|
| `make` | Builds and tests the image (`make all`) |
| `make build` | Builds the container image |
| `make test` | Runs an interactive test container |
| `make save` | Saves the image as a `.tar.gz` file |
| `make load` | Loads the image from `.tar.gz` |
| `make clean` | Deletes the image from the system |


---

## **Final Thoughts**  

Using **GNU Make** with **Docker** (or **Podman**) provides a structured, repeatable, and automated way to manage container builds and executions. By defining targets in a `Makefile`, you simplify commands and ensure consistency in your development workflow.  


🚀 Happy coding, and may your containers always build successfully! 🎯
