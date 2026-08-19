# Automated Deployment with Docker and GitHub Actions

Deployment can also be automated with Docker and GitHub Actions. What makes GitHub special is that we can create an image and push it into the GitHub Container Registry (GHCR).

> **Note:** Remember to add permissions at the workflow level.

```yaml
permissions:
  contents: read
  packages: write

jobs:
  deployment: 
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Code Checkout
        uses: actions/checkout@v4
      
      - name: Login to GHCR
        run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      
      - name: Build Docker Image
        working-directory: 7-jobs-and-docker-container/starting-project
        run: docker build -t ghcr.io/ziancarlos/custom-app:${{ github.sha }} .
      
      - name: Push Docker Image
        working-directory: 7-jobs-and-docker-container/starting-project
        run: docker push ghcr.io/ziancarlos/custom-app:${{ github.sha }}
```

## Next Steps

These are the steps after pushing. Compose just has to use the latest image from GHCR. By doing that, we can SSH into the VPS and just `docker compose up` our apps without having to rebuild the apps themselves.
