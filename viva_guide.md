# DevOps Viva Guide: Docker & GitHub Actions

This document explains every single file and line of code created to dockerize the project and automate it using GitHub Actions. It is designed to help you prepare for your DevOps viva.

---

## 1. `requirements.txt`

This file lists the Python packages needed to run your application. Docker will read this file to know what to install.

```text
feedparser
python-docx
requests
beautifulsoup4
python-dateutil
```
**Explanation:**
- These are simply the names of the third-party libraries your script `news_scraper.py` imports. When `pip install -r requirements.txt` is run, it fetches and installs these exact packages.

---

## 2. `Dockerfile`

The `Dockerfile` is a blueprint for building your Docker image. An image is a standalone, executable package that includes everything needed to run your software (code, runtime, system tools, libraries).

```dockerfile
FROM python:3.11-slim
```
- **`FROM`**: This instruction sets the Base Image for subsequent instructions. We use `python:3.11-slim`, which is a lightweight Linux distribution with Python 3.11 pre-installed. "slim" means it doesn't include unnecessary tools, keeping the image size small.

```dockerfile
WORKDIR /app
```
- **`WORKDIR`**: This sets the working directory inside the container. Any subsequent `COPY`, `RUN`, or `CMD` instructions will be executed in this `/app` folder.

```dockerfile
COPY requirements.txt .
```
- **`COPY`**: This copies the `requirements.txt` from your local machine (the host) into the current working directory (`.`, which is `/app`) inside the container.

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```
- **`RUN`**: This executes a command during the image build process.
- **`pip install -r requirements.txt`**: Installs the dependencies.
- **`--no-cache-dir`**: This flag tells pip not to save the downloaded packages in a cache directory. This is a best practice in Docker to keep the final image size as small as possible.

```dockerfile
COPY . .
```
- **`COPY . .`**: This copies the rest of your local project files (the first `.`) into the container's working directory (the second `.`). We do this *after* installing requirements so that if we change our code (but not our requirements), Docker doesn't have to reinstall all packages. This leverages Docker's layer caching mechanism.

```dockerfile
CMD ["python", "news_scraper.py"]
```
- **`CMD`**: This specifies the default command to run when the container starts. Here, it tells the container to execute `python news_scraper.py`.

---

## 3. `.dockerignore`

This file works exactly like `.gitignore`. It tells Docker which files and directories on your local machine should *not* be copied into the Docker image when running `COPY . .`.

```text
.git
.github
reports/
__pycache__/
*.pyc
.env
venv/
env/
```
**Explanation:**
- **`.git` / `.github`**: Version control files aren't needed to run the app.
- **`reports/`**: We don't want to copy existing local reports into the image; the container will generate new ones.
- **`__pycache__` / `*.pyc`**: Python bytecode files. They are generated automatically and copying them can cause issues if your local OS differs from the container's OS.
- **`.env` / `venv/`**: Local environment variables and virtual environments should not be copied.

---

## 4. `.github/workflows/scrape.yml`

This is the GitHub Actions workflow file. It defines a CI/CD (Continuous Integration / Continuous Deployment) pipeline. It uses YAML format.

```yaml
name: Daily News Scraper
```
- **`name`**: The name of the workflow as it will appear in the GitHub Actions tab.

```yaml
on:
  push:
    branches:
      - main
  schedule:
    - cron: '0 8 * * *'
  workflow_dispatch:
```
- **`on`**: Defines the events that trigger the workflow.
- **`push:`**: Triggers when someone pushes code to the `main` branch.
- **`schedule:`**: Uses a cron expression (`'0 8 * * *'`) to run automatically. This translates to "minute 0, hour 8, every day of the month, every month, every day of the week" (8:00 AM UTC).
- **`workflow_dispatch:`**: This allows you to manually click a "Run workflow" button in the GitHub UI.

```yaml
jobs:
  scrape-news:
    runs-on: ubuntu-latest
```
- **`jobs:`**: A workflow consists of one or more jobs. We have one job named `scrape-news`.
- **`runs-on: ubuntu-latest`**: This tells GitHub to provision a fresh, temporary virtual machine running the latest version of Ubuntu Linux to execute our steps.

```yaml
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
```
- **`steps:`**: The sequence of tasks within the job.
- **`uses: actions/checkout@v4`**: This step uses a pre-built GitHub Action to clone (checkout) your repository's code onto the runner VM so the workflow can access it.

```yaml
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
```
- **`uses: docker/setup-buildx-action@v3`**: Sets up Docker Buildx, an advanced Docker build tool that allows for more features (though standard Docker build would also work, this is a modern best practice).

```yaml
      - name: Build Docker image
        run: docker build -t news-scraper:latest .
```
- **`run:`**: Executes a command line script.
- **`docker build -t news-scraper:latest .`**: Builds the Docker image from our `Dockerfile`. `-t` tags (names) the image as `news-scraper:latest`. `.` specifies the current directory as the build context.

```yaml
      - name: Run Scraper in Docker
        run: |
          mkdir -p reports
          docker run --rm -v ${{ github.workspace }}/reports:/app/reports news-scraper:latest
```
- **`mkdir -p reports`**: Ensures a `reports` directory exists on the GitHub runner VM.
- **`docker run`**: Starts a container from the image we just built.
- **`--rm`**: Tells Docker to automatically remove the container when it finishes running to save space.
- **`-v ${{ github.workspace }}/reports:/app/reports`**: This is a Volume Mount. It maps the `reports` folder on the host (the GitHub runner VM) to the `/app/reports` folder inside the container. This way, when the Python script saves files to `/app/reports`, they are actually saved onto the runner VM, surviving the container's destruction.

```yaml
      - name: Upload Reports as Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: news-reports
          path: reports/
          retention-days: 7
```
- **`uses: actions/upload-artifact@v4`**: Uses a pre-built Action to upload files from the runner VM to GitHub.
- **`name: news-reports`**: The name of the downloadable zip file that will be created.
- **`path: reports/`**: The directory to upload. Because of our volume mount in the previous step, this directory contains the fresh `.docx` and `.html` files generated by the container.
- **`retention-days: 7`**: Tells GitHub to automatically delete these artifacts after 7 days to save storage space.
