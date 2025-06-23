# simple-ecommerce-website-nodejs
*This application is an e-commerce website application. The main software language is `Node.js` and `express`, `express-session`, `express-handlebars`, `body-parser`, `mongodb` have been used. On the database side, `MongoDB` is preferred.*
# 🚀 Node.js CI/CD Pipeline with GitHub Actions, SonarQube, Nexus, and AWS EC2

This project showcases a complete DevOps pipeline to **build, test, analyze, store, and deploy** a Node.js application using modern tools like **GitHub Actions**, **SonarQube**, **Nexus Repository**, **Nginx**, and **AWS EC2**.

---

## 📌 Architecture Overview

Developer → VS Code → Git → GitHub  
        ↓  
GitHub Actions CI/CD Pipeline:  
  - 🔍 SonarQube → PostgreSQL  
  - 📦 Nexus Repository  
  - 🚀 Deploy via Nginx → Node.js app on EC2

---

## 🔧 Tools Used

| Tool                 | Purpose                                      |
|----------------------|----------------------------------------------|
| GitHub               | Source code management and CI/CD trigger     |
| GitHub Actions       | Automates build, test, analysis, deployment  |
| SonarQube            | Static code analysis                         |
| PostgreSQL           | Stores SonarQube analysis results            |
| Nexus Repository Pro | Stores build artifacts                       |
| Nginx                | Reverse proxy for secure traffic routing     |
| Amazon EC2           | Hosts the Node.js application                |
| Node.js              | Application runtime                          |

---

## 🔁 CI/CD Workflow

1. Developer pushes code to GitHub
2. GitHub Actions pipeline runs:
   - Checks out the code
   - Runs SonarQube for code quality checks
   - Stores analysis in PostgreSQL
   - Uploads build artifacts to Nexus
   - Deploys to EC2 behind Nginx reverse proxy

---

## 🧪 SonarQube Integration

- Performs static analysis on Node.js code
- Checks for:
  - Bugs
  - Vulnerabilities
  - Code smells
- Results stored in PostgreSQL (SonarQube backend DB)

---

## 📦 Artifact Management (Nexus)

- Build artifacts (e.g., zipped app) are stored in Nexus Repository
- Useful for future deployments, rollback, or tracking builds

---

## 🌐 Deployment to EC2

- Nginx on EC2 acts as a reverse proxy
- Traffic forwarded to the Node.js app
- Node app may be run via `pm2` or `systemd`

---

## ✅ Prerequisites

- AWS EC2 instance set up with:
  - Node.js
  - Nginx configured
  - SSH access or GitHub runner installed
- SonarQube server running (local or cloud)
- PostgreSQL backend for SonarQube
- Nexus Repository (local or cloud)
- AWS credentials or secrets added to GitHub Actions

---

## 🚀 Getting Started

1. Clone this repository
name: Deploy Node.js App to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Configure npm to use Nexus for install
        run: |
          echo "//$(echo ${{ secrets.NEXUS_REGISTRY }} | sed 's|https\?://||'):_auth=$(echo -n '${{ secrets.NEXUS_USERNAME }}:${{ secrets.NEXUS_PASSWORD }}' | base64)" > .npmrc
          echo "registry=${{ secrets.NEXUS_REGISTRY }}" >> .npmrc

      - name: Install Dependencies from Nexus
        run: npm install

      - name: Configure npm to publish to Nexus
        run: |
          echo "//$(echo ${{ secrets.NEXUS_PUBLISH_REGISTRY }} | sed 's|https\?://||'):_auth=$(echo -n '${{ secrets.NEXUS_USERNAME }}:${{ secrets.NEXUS_PASSWORD }}' | base64)" > .npmrc
          echo "registry=${{ secrets.NEXUS_PUBLISH_REGISTRY }}" >> .npmrc

      - name: Publish to Nexus Repository
        run: npm publish --registry=${{ secrets.NEXUS_PUBLISH_REGISTRY }}

      - name: Run SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@v5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

      # Optional: Quality Gate check
      # - name: Check Quality Gate Status
      #   uses: SonarSource/sonarqube-quality-gate-action@v1
      #   timeout-minutes: 5
      #   env:
      #     SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

      - name: Deploy to EC2
        env:
          HOST: ${{ secrets.SSH_HOST }}
          KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        run: |
          echo "$KEY" > key.pem
          chmod 600 key.pem
          rsync -avz -e "ssh -i key.pem -o StrictHostKeyChecking=no" ./ ec2-user@$HOST:/home/ec2-user/simple-ecommerce/
          ssh -i key.pem ec2-user@$HOST << 'EOF'
            cd ~/simple-ecommerce
            echo "//$(echo ${{ secrets.NEXUS_REGISTRY }} | sed 's|https\?://||'):_auth=$(echo -n '${{ secrets.NEXUS_USERNAME }}:${{ secrets.NEXUS_PASSWORD }}' | base64)" > .npmrc
            echo "registry=${{ secrets.NEXUS_REGISTRY }}" >> .npmrc
            npm install
            pm2 stop all || true
            pm2 start app.js --name ecommerce
            pm2 save
          EOF
this is the yaml code i used           
## Architecture
![image](https://github.com/user-attachments/assets/7cbf4eca-5539-4005-a671-80fdb7fceef8)
## Succesfull deploy
![image](https://github.com/user-attachments/assets/05795a16-e499-441e-86e5-e168cd3d1314)


## Sample Screenshots
![home](https://github.com/eroldmrclk/simple-ecommerce-website-nodejs/blob/master/images/home.png)
![admin-login](https://github.com/eroldmrclk/simple-ecommerce-website-nodejs/blob/master/images/admin-login.png)
## 📣 Author

**Ashik Antony**  
🔗 https://www.linkedin.com/in/ashikantony  
✍️ Blog: https://ashikantony.hashnode.dev
