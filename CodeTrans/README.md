# Code Translation Application

Code translation is the process of converting code written in one programming language to another programming language while maintaining the same functionality. This process is also known as code conversion, source-to-source translation, or transpilation. Code translation is often performed when developers want to take advantage of new programming languages, improve code performance, or maintain legacy systems. Some common examples include translating code from Python to Java, or from JavaScript to TypeScript.

The workflow falls into the following architecture:

![architecture](./assets/img/code_trans_architecture.png)

The CodeTrans example is implemented using the component-level microservices defined in [GenAIComps](https://github.com/opea-project/GenAIComps). The flow chart below shows the information flow between different microservices for this example.

```mermaid
---
config:
  flowchart:
    nodeSpacing: 400
    rankSpacing: 100
    curve: linear
  themeVariables:
    fontSize: 50px
---
flowchart LR
    %% Colors %%
    classDef blue fill:#ADD8E6,stroke:#ADD8E6,stroke-width:2px,fill-opacity:0.5
    classDef orange fill:#FBAA60,stroke:#ADD8E6,stroke-width:2px,fill-opacity:0.5
    classDef orchid fill:#C26DBC,stroke:#ADD8E6,stroke-width:2px,fill-opacity:0.5
    classDef invisible fill:transparent,stroke:transparent;
    style CodeTrans-MegaService stroke:#000000

    %% Subgraphs %%
    subgraph CodeTrans-MegaService["CodeTrans MegaService "]
        direction LR
        LLM([LLM MicroService]):::blue
    end
    subgraph UserInterface[" User Interface "]
        direction LR
        a([User Input Query]):::orchid
        UI([UI server<br>]):::orchid
    end


    LLM_gen{{LLM Service <br>}}
    GW([CodeTrans GateWay<br>]):::orange
    NG([Nginx MicroService]):::blue


    %% Questions interaction
    direction LR
    NG <==> UserInterface
    a[User Input Query] --> UI
    UI --> GW
    GW <==> CodeTrans-MegaService


    %% Embedding service flow
    direction LR
    LLM <-.-> LLM_gen

```

This Code Translation use case demonstrates Text Generation Inference across multiple platforms. Currently, we provide examples for [Intel Gaudi2](https://www.intel.com/content/www/us/en/products/details/processors/ai-accelerators/gaudi-overview.html) and [Intel Xeon Scalable Processors](https://www.intel.com/content/www/us/en/products/details/processors/xeon.html), and we invite contributions from other hardware vendors to expand OPEA ecosystem.

## Deploy Code Translation Service

The Code Translation service can be effortlessly deployed on either Intel Gaudi2 or Intel Xeon Scalable Processor.

Currently we support two ways of deploying Code Translation services on docker:

1. Start services using the docker image on `docker hub`:

   ```bash
   docker pull opea/codetrans:latest
   ```

2. Start services using the docker images `built from source`: [Guide](./docker_compose/intel/cpu/xeon/README.md)

### Required Models

By default, the LLM model is set to a default value as listed below:

| Service | Model                              |
| ------- | ---------------------------------- |
| LLM     | mistralai/Mistral-7B-Instruct-v0.3 |

Change the `LLM_MODEL_ID` in `docker_compose/set_env.sh` for your needs.

### Setup Environment Variable

To set up environment variables for deploying Code Translation services, follow these steps:

1. Set the required environment variables:

   ```bash
   # Example: host_ip="192.168.1.1"
   export host_ip="External_Public_IP"
   # Example: no_proxy="localhost, 127.0.0.1, 192.168.1.1"
   export no_proxy="Your_No_Proxy"
   export HUGGINGFACEHUB_API_TOKEN="Your_Huggingface_API_Token"
   # Example: NGINX_PORT=80
   export NGINX_PORT=${your_nginx_port}
   ```

2. If you are in a proxy environment, also set the proxy-related environment variables:

   ```bash
   export http_proxy="Your_HTTP_Proxy"
   export https_proxy="Your_HTTPs_Proxy"
   ```

3. Set up other environment variables:

   ```bash
   source ./docker_compose/set_env.sh
   ```

### Deploy with Docker

#### Deploy Code Translation on Gaudi

Find the corresponding [compose.yaml](./docker_compose/intel/hpu/gaudi/compose.yaml).

```bash
cd GenAIExamples/CodeTrans/docker_compose/intel/hpu/gaudi
docker compose up -d
```

Refer to the [Gaudi Guide](./docker_compose/intel/hpu/gaudi/README.md) to build docker images from source.

#### Deploy Code Translation on Xeon

Find the corresponding [compose.yaml](./docker_compose/intel/cpu/xeon/compose.yaml).

```bash
cd GenAIExamples/CodeTrans/docker_compose/intel/cpu/xeon
docker compose up -d
```

Refer to the [Xeon Guide](./docker_compose/intel/cpu/xeon/README.md) for more instructions on building docker images from source.

### Deploy CodeTrans on Kubernetes using Helm Chart

Refer to the [CodeTrans helm chart](./kubernetes/helm/README.md) for instructions on deploying CodeTrans on Kubernetes.

## Consume Code Translation Service

Two ways of consuming Code Translation Service:

1. Use cURL command on terminal

   ```bash
   curl http://${host_ip}:7777/v1/codetrans \
       -H "Content-Type: application/json" \
       -d '{"language_from": "Golang","language_to": "Python","source_code": "package main\n\nimport \"fmt\"\nfunc main() {\n    fmt.Println(\"Hello, World!\");\n}"}'
   ```

2. Access via frontend

To access the frontend, open the following URL in your browser: http://{host_ip}:5173.

By default, the UI runs on port 5173 internally.

## Troubleshooting

1. If you get errors like "Access Denied", [validate micro service](https://github.com/opea-project/GenAIExamples/tree/main/CodeTrans/docker_compose/intel/cpu/xeon/README.md#validate-microservices) first. A simple example:

   ```bash
   http_proxy=""
   curl http://${host_ip}:8008/generate \
     -X POST \
     -d '{"inputs":"    ### System: Please translate the following Golang codes into  Python codes.    ### Original codes:    '\'''\'''\''Golang    \npackage main\n\nimport \"fmt\"\nfunc main() {\n    fmt.Println(\"Hello, World!\");\n    '\'''\'''\''    ### Translated codes:","parameters":{"max_tokens":17, "do_sample": true}}' \
     -H 'Content-Type: application/json'
   ```

2. (Docker only) If all microservices work well, check the port ${host_ip}:7777, the port may be allocated by other users, you can modify the `compose.yaml`.

3. (Docker only) If you get errors like "The container name is in use", change container name in `compose.yaml`.
