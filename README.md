## K8sGPT + Minikube + Ollama

<img src=cover.png>

### 1. Start Minikube

**macOS**

```bash
brew install minikube
minikube start
kubectl get nodes
```

`minikube start` creates a local Kubernetes cluster. `kubectl get nodes` confirms the cluster is up and the node is Ready.

**Ubuntu**

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

minikube start
kubectl get nodes
```

`minikube start` creates a local Kubernetes cluster. `kubectl get nodes` confirms the cluster is up and the node is Ready.

> For ARM Ubuntu, replace `amd64` with `arm64`.

---

### 2. Install K8sGPT

**macOS**

```bash
brew tap k8sgpt-ai/k8sgpt
brew install k8sgpt
k8sgpt version
```

`k8sgpt version` prints the installed K8sGPT version so you know the CLI is on your PATH.

**Ubuntu**

```bash
curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/latest/download/k8sgpt_amd64.deb
sudo dpkg -i k8sgpt_amd64.deb

k8sgpt version
```

`k8sgpt version` prints the installed K8sGPT version so you know the CLI is on your PATH.

> For ARM Ubuntu, use the `arm64` package.

---

### 3. Install Ollama

**macOS**

```bash
brew install ollama
```

Start it:

```bash
ollama serve
```

`ollama serve` starts the local LLM server so K8sGPT can call it.

**Ubuntu**

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Start/check it:

```bash
sudo systemctl enable --now ollama
```

Starts the Ollama service and enables it to run on boot.

---

### 4. Download the LLM

Same on both:

```bash
ollama pull llama3.1:8b
```

Downloads the Llama 3.1 8B model so Ollama can run it locally.

Check:

```bash
ollama list
```

Shows which models are already downloaded.

Ollama = engine 🏎️
Llama 3.1 = model that runs on the engine 🧠

## So you need to install Ollama first, then pull the model.

### 5. Connect K8sGPT → Ollama

Same on both:

```bash
k8sgpt auth add \
  --backend localai \
  --model llama3.1:8b \
  --baseurl http://localhost:11434/v1
```

Registers Ollama as K8sGPT's AI backend: local OpenAI-compatible API on port 11434, using the Llama 3.1 8B model.

---

### 6. Create a broken deployment

```bash
kubectl create deployment broken-app \
  --image=nginx:doesnotexist

kubectl get pods
```

Creates a Deployment that tries to pull a fake image, so the pod fails. `kubectl get pods` shows the failing pod (ImagePullBackOff).

Expected:

```text
ImagePullBackOff
```

---

### 7. Let K8sGPT analyze it

First without AI:

```bash
k8sgpt analyze
```

Scans the cluster for problems and reports them without an LLM explanation.

Then with Ollama:

```bash
k8sgpt analyze --explain --backend localai --namespace default
```

Same scan, but Ollama explains _why_ things are broken and what to do, using the `localai` backend in the `default` namespace.

### Your video flow

**Install → Minikube → Ollama → K8sGPT → break Kubernetes → K8sGPT diagnoses it → fix it → analyze again.**
