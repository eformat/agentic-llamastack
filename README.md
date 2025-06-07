# agentic-llamastack

LLamastack agentic demos.

## Pre-requisites

- RHOAI
- LLM being served with OpenAI endpoint

## Setup

Create project

```bash
oc new-proect llama-serve
```

Deploy MCP Servers `openshift-mcp`, `github-mcp`

```bash
kustomize build applications/mcp-servers/openshift-mcp | oc apply -n llama-serve -f-
```

```bash
export GITHUB_TOKEN=ghp_your_github_token
cat applications/mcp-servers/github-mcp/github-secret.yaml | envsubst | oc apply -f-
kustomize build applications/mcp-servers/github-mcp | oc apply -n llama-serve -f-
```

Deploy LLamaStack. 

You will need to customize the overlay [applications/llama-stack/overlays/sno/deployment.yaml](applications/llama-stack/overlays/sno/deployment.yaml) to point to your LLM endpoints.

Also check [applications/llama-stack/base/run.yaml](applications/llama-stack/base/run.yaml) for the LLamaStack configurations which generate a ConfigMap.

```bash
kustomize build applications/llama-stack/overlays/sno | oc apply -n llama-serve -f-
```

## Notebooks

To run the notebooks.

```bash
cd notebooks
pip install -r requirements.txt
```

Environment - update to suit, here i port-forward Services from OpenShift to develop locally, but you could run this all in RHOAI for a demo.

```bash
oc -n llama-serve port-forward svc/llamastack-server 8321:8321 &
oc -n llama-serve port-forward svc/ocp-mcp-server 8080:8000 &
oc -n llama-serve port-forward svc/github-mcp-server-with-rh-nodejs 8081:80 &
```

Export env vars

```bash
# mandatory unless a local deployment is used
export REMOTE_BASE_URL="http://localhost:8321"
# for the agentic/tool demos
export TAVILY_SEARCH_API_KEY="tvly-dev-your-key"
# optional environment variables for all demos
export TEMPERATURE="0.0"
export TOP_P="0.95"
export MAX_TOKENS="15000"
export STREAM="True"
# for the RAG demos only - mandatory
export VDB_PROVIDER="milvus"
export VDB_EMBEDDING="all-MiniLM-L6-v2"
# for the RAG demos only - optional
export VDB_EMBEDDING_DIMENSION="384"
export VECTOR_DB_CHUNK_SIZE="512"
export USE_PROMPT_CHAINING="True"
# for the eval notebook only - mandatory
export LLM_AS_JUDGE_MODEL_ID=""
# MCP-related
export REMOTE_OCP_MCP_URL="http://localhost:8080/sse"
export REMOTE_GITHUB_MCP_URL="http://localhost:8081/sse"
#export REMOTE_SLACK_MCP_URL="http://slack-mcp-server:80/sse"
```

Code

```bash
code .
```
