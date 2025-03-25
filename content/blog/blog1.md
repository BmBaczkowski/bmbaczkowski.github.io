---
title: "Containerized Writing: Ensuring Manuscript Reproducibility with Quarto & Docker" 
date: 2025-02-12
lastmod: 2025-02-12
tags: ["Quarto", "docker", "podman", "reproducibility", "apaquarto"]
author: ["Blazej M. Baczkowski"]
description: ""
summary: "How to use Quarto in a Docker container?"
editPost:
    URL: ""
    Text: ""
showToc: true
disableAnchoredHeadings: false

---

Reproducibility is crucial in research and data science. One way to achieve it is by using containers, ensuring that your environment remains consistent across different systems.  

While this approach is commonly used in data analysis, I also enjoy writing my manuscripts in a fully reproducible and containerized way. It ensures consistency across environments, makes collaboration easier, and—let’s be honest—there’s something satisfying about having a perfectly isolated setup for writing and compiling documents. 😄

This guide walks you through creating a **Docker image** with [**Quarto**](https://quarto.org/) inside a container.  

If you're working on a manuscript in APA style, check out [**apaquarto**](https://wjschne.github.io/apaquarto/) -- a package for writing documents in APA format with Quarto.

---

## **Step-by-Step: Building a Docker Image with Quarto**  

### **1. Choose a Base Image**  

We'll use `rocker/rstudio:4.3.2`, a pre-built image that includes **RStudio Server** and **R 4.3.2**, and supports **ARM architecture**.
[**Rocker**](https://rocker-project.org/) is an open-source project that provides optimized Docker images for R development.  

```dockerfile
FROM rocker/rstudio:4.3.2
```

---

### **2. Install System Dependencies**  

To install Quarto, we first need some system utilities:  

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    gdebi-core
```

---

### **3. Install R Packages with `pak`**  

[`pak`](https://pak.r-lib.org/) is a modern R package manager designed for fast and reliable dependency installation.  

```dockerfile
RUN R -q -e 'install.packages("pak", repos=sprintf("https://r-lib.github.io/p/pak/stable/%s/%s/%s", .Platform$pkgType, R.Version()$os, R.Version()$arch))'
```

For project-specific R packages, I keep a list in `r-pkgs.txt`, where each line contains a package name and version:  

```plain
# Example: r-pkgs.txt  
easystats@0.7.2  
extraDistr@1.10.0  
lme4@1.1-35.5  
MASS@7.3-60.2  
```

We copy this list into the container and install the packages using `pak`:  

```dockerfile
COPY r-pkgs.txt /tmp/packages.txt  
RUN R -q -e 'pak::pkg_install(pkg=readLines("/tmp/packages.txt"))'
```

---

### **4. Install TeX Live (for LaTeX & PDF Generation)**  

To generate PDFs, we install a **LaTeX** distribution:  

```dockerfile
RUN /rocker_scripts/install_texlive.sh
```

Then, we ensure TeX Live’s binary directory is available:  

```dockerfile
ENV PATH="$PATH:/usr/local/texlive/bin/linux"
```

Like R packages, I keep a separate list for LaTeX packages in `tex-pkgs.txt`:  

```plain
# Example: tex-pkgs.txt  
amsfonts  
amsmath  
apa7  
```

We copy and install them using the TeX Live package manager (`tlmgr`):  

```dockerfile
COPY tex-pkgs.txt /tmp/  
RUN tlmgr install $(cat /tmp/tex-pkgs.txt)  
RUN R -q -e 'tinytex::r_texmf(action="add")'
```

---

### **5. Install Quarto**  

We download and install a specific Quarto version for reproducibility:  

```dockerfile
RUN ARCH=$(dpkg --print-architecture) && \
    curl -LO https://github.com/quarto-dev/quarto-cli/releases/download/v1.6.42/quarto-1.6.42-linux-${ARCH}.deb && \
    gdebi --non-interactive quarto-1.6.42-linux-${ARCH}.deb
```

---

### **6. Cleanup (Reduce Image Size)**  

To keep the image lightweight, we remove unnecessary package lists:  

```dockerfile
RUN rm -rf /var/lib/apt/lists/*
```

---

### **7. Set the Working Directory**  

Finally, we define a working directory inside the container for convenience:  

```dockerfile
WORKDIR /home/dockeruser
```  

---

## **Final Thoughts**  

By using this approach, you create a **fully reproducible** Quarto environment inside Docker. This setup ensures consistency across different machines, making it perfect for scientific writing, publishing, and collaboration. 🚀  

---


## **Full dockerfile**

```dockerfile
# Use a stable Rocker image as a base
FROM rocker/rstudio:4.3.2

# Update the package list and install necessary system dependencies
# - curl: Used for downloading files from URLs
# - gdebi-core: Required for installing .deb packages (like Quarto)
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    gdebi-core

# Install the 'pak' package manager for R
RUN R -q -e 'install.packages("pak", repos=sprintf("https://r-lib.github.io/p/pak/stable/%s/%s/%s", .Platform$pkgType, R.Version()$os, R.Version()$arch))'

# Copy the list of required R packages into the container
COPY r-pkgs.txt /tmp/

# Install the R packages listed in 'r-pkgs.txt' using pak
RUN R -q -e 'pak::pkg_install(pkg=readLines("/tmp/packages.txt"))'

# Install TeX Live for LaTeX document support
RUN /rocker_scripts/install_texlive.sh

# Update PATH to include TeX Live binaries
ENV PATH="$PATH:/usr/local/texlive/bin/linux"

# Copy the list of TeX packages into the container
COPY tex-pkgs.txt /tmp/

# Install required TeX packages
RUN tlmgr install `cat /tmp/tex-pkgs.txt`

# Ensure R recognizes the TeX installation
RUN R -q -e 'tinytex::r_texmf(action="add")'

# Determine system architecture, download, and install Quarto
RUN ARCH=$(dpkg --print-architecture) && \
    curl -LO https://github.com/quarto-dev/quarto-cli/releases/download/v1.6.42/quarto-1.6.42-linux-${ARCH}.deb && \
    gdebi --non-interactive quarto-1.6.42-linux-${ARCH}.deb

# Clean up package lists to reduce image size
RUN rm -rf /var/lib/apt/lists/* 

# Set the working directory
WORKDIR /home/dockeruser
```





