README By ChatGPT, partially reviewed by myself!

⸻

🚀 Amplify Node.js Build Optimization with Prebuilt Docker Images

📌 Overview

This repository demonstrates a practical optimization technique for AWS Amplify Node.js builds.

Instead of running:

npm ci

during every Amplify build — and relying on Amplify’s cache — this project uses:
	•	🐳 A prebuilt Docker image
	•	⚙️ GitHub Actions
	•	📦 AWS ECR
	•	🔁 Trigger only when package-lock.json changes

The result:

Before	After
⏱ 11–12 minutes	⏱ 5–6 minutes
💸 Higher Amplify cost	💰 Reduced build cost


⸻

🎯 The Idea

Amplify runs npm ci during every build, which:
	•	Downloads dependencies
	•	Installs node_modules
	•	Takes several minutes
	•	Consumes build minutes (cost)

Instead of installing dependencies inside Amplify:

✅ We pre-build an image containing node_modules
✅ Push it to AWS ECR
✅ Amplify build reuses the pre-installed dependencies

This removes dependency installation from the Amplify build phase.

⸻

🏗 Architecture

GitHub Repository
        │
        ├── Dockerfile.build
        ├── package.json
        ├── package-lock.json
        │
        ▼
GitHub Actions
(.github/workflows/amplify-build-image.yml)
        │
        │  If package-lock.json changed
        ▼
Build Docker Image
        │
        │  npm ci executed HERE
        ▼
Push Image to AWS ECR
        │
        ▼
AWS Amplify Build
(amplify.yaml)
        │
        │ Pull latest ECR image
        │ Reuse node_modules
        ▼
Faster Build 🚀


⸻

📂 Key Files Explained

🐳 Dockerfile.build

Builds a Node.js image and installs dependencies:

FROM node:20-alpine

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

CMD ["npm", "run", "build"]

Dependencies are installed once inside this image.

⸻

⚙️ .github/workflows/amplify-build-image.yml

GitHub Actions workflow that:
	•	Triggers when package-lock.json changes
	•	Builds the Docker image
	•	Pushes the image to AWS ECR

This ensures dependencies are rebuilt only when needed.

⸻

📦 package.json / package-lock.json

package-lock.json is the trigger point.

When it changes:
	•	New dependencies → new Docker image
	•	No changes → no rebuild → Amplify reuses previous image

⸻

🏗 amplify.yaml

Amplify build configuration.

Instead of running npm ci, the build:
	•	Pulls the latest image from ECR
	•	Uses pre-installed node_modules
	•	Proceeds directly to the build step

Example idea:

preBuild:
  commands:
    - docker pull <your-ecr-image>:latest

build:
  commands:
    - npm run build

No dependency installation during Amplify build.

⸻

📊 Why This Works

Default Amplify Flow

npm ci → 5-6 minutes
build → 4-5 minutes
Total: 11-12 minutes

Optimized Flow

Pull image → ~30 seconds
build → 4-5 minutes
Total: 5-6 minutes


⸻

💰 Cost Optimization Impact

Amplify billing is based on build minutes.

Cutting build time in half:
	•	Reduces build cost
	•	Reduces CI time
	•	Speeds up deployments
	•	Improves developer experience

⸻

🧠 When to Use This

This strategy is ideal when:
	•	You have heavy Node.js dependencies
	•	Builds run frequently
	•	npm ci is the biggest bottleneck
	•	You want deterministic builds based on package-lock.json

⸻

⚠️ Trade-offs
	•	Slightly more complex CI setup
	•	Requires ECR access
	•	Docker image maintenance

But for active projects, the time savings are significant.

⸻

🧪 How to Reproduce
	1.	Configure AWS ECR repository
	2.	Add GitHub secrets for:
	•	AWS_ACCESS_KEY_ID
	•	AWS_SECRET_ACCESS_KEY
	•	AWS_REGION
	3.	Configure .github/workflows/amplify-build-image.yml
	4.	Modify amplify.yaml to pull image
	5.	Push changes
	6.	Observe reduced Amplify build times

⸻

📢 LinkedIn Post Context

This repository supports the following post:

“While having fun developing my new personal full stack (AWS Amplify) app, one trick I found to save money on NodeJS builds is to avoid doing npm ci in Amplify and instead build a Docker image whenever package-lock.json changes. By doing this, I reduced Amplify build time from 11–12 minutes to 5–6 minutes.”

⸻

🛠 Future Improvements
	•	Multi-stage Docker builds
	•	Layer caching improvements
	•	Separate dev/prod images
	•	pnpm support
	•	TurboRepo support

⸻

🤝 Contributions

This is a demonstration repository for build optimization strategy sharing.

Feel free to fork and adapt.
