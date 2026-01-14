# LLB Benchmarking with GenAI-Perf

Clone the Repo and edit the Ansible Inventory file

```
cat llm-labs/llmops/ansible/inventory/hosts.ini
; [vllm_server]
; server_name ansible_user=ubuntu
[llm_workers]
<IP Address> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/<your_key_file>
```

Run the Playbook

```
ansible-playbook -i ansible/inventory/hosts.ini ansible/setup_worker.yml
```
Create the Docker external network to be used by all containers

```
docker network create llmops-net
```
Launch the Containers

```
docker compose -f docker-compose-vllm-qwen3-0.6B.yml up -d
docker compose -f docker-compose.monitoring.yml up -d
```
Install GenAI-Perf

```
pip install genai-perf
```

Run the load with different concurrency

```
for c in 64 96 128 192 256; do
  genai-perf profile \
    -m Qwen/Qwen3-0.6B \
    --endpoint-type chat \
    --synthetic-input-tokens-mean 200 --synthetic-input-tokens-stddev 0 \
    --output-tokens-mean 100 --output-tokens-stddev 0 \
    --request-count 400 \
    --warmup-request-count 10 \
    --tokenizer Qwen/Qwen3-0.6B \
    --concurrency $c
done
```
Output would look like

```
 NVIDIA GenAI-Perf | LLM Metrics
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┓
┃                            Statistic ┃      avg ┃      min ┃      max ┃      p99 ┃      p90 ┃      p75 ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━┩
│                 Request Latency (ms) │ 2,216.66 │ 1,467.06 │ 2,708.24 │ 2,708.00 │ 2,702.71 │ 2,637.29 │
│      Output Sequence Length (tokens) │    99.98 │    99.00 │   101.00 │   100.00 │   100.00 │   100.00 │
│       Input Sequence Length (tokens) │   200.00 │   200.00 │   200.00 │   200.00 │   200.00 │   200.00 │
│ Output Token Throughput (tokens/sec) │ 9,580.77 │      N/A │      N/A │      N/A │      N/A │      N/A │
│         Request Throughput (per sec) │    95.83 │      N/A │      N/A │      N/A │      N/A │      N/A │
│                Request Count (count) │   400.00 │      N/A │      N/A │      N/A │      N/A │      N/A │
└──────────────────────────────────────┴──────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```
