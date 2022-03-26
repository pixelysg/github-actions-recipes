# Deploy to Google Cloud App Engine

This GitHub Actions workflow will automatically deploy your `npm` app onto Google Cloud App Engine, including deploy previews and deploy notifications.

## Prerequisites

- Access to existing project on Google Cloud
- App Engine already initialised through the console
- App Engine Admin API and Cloud Storage API enabled
- Able to create a a service account key and configure in your repository as `GCP_SA_KEY`

## Usage

```yml
name: Deploy to App Engine
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - id: 'checkout'
      name: Checkout
      uses: actions/checkout@v2
    - id: 'auth'
      name: 'Authenticate'
      uses: 'google-github-actions/auth@v0'
      with:
        credentials_json: '${{ secrets.GCP_SA_KEY }}'
    - id: 'npm-ci'
      name: 'Install npm dependencies'
      run: npm ci
    - id: 'npm-run-build'
      name: 'Build app'
      run: npm run build

    - if: github.ref == 'refs/heads/main'
      id: 'deploy'
      name: 'Deploy to App Engine'
      uses: 'google-github-actions/deploy-appengine@v0'
      with:
        deliverables: app.yaml

    - if: github.ref != 'refs/heads/main'
      id: 'deploy-preview'
      name: 'Deploy Preview to App Engine'
      uses: 'google-github-actions/deploy-appengine@v0'
      with:
        deliverables: app.yaml
        promote: false

    - id: 'Comment'
      name: 'Comment'
      uses: peter-evans/commit-comment@v1
      with:
        body: |
          🥂 Deployed! Browse the preview: ${{ steps.deploy.outputs.url || steps.deploy-preview.outputs.url }}
```
