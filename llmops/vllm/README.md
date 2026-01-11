# Nvidia GenAI-Perf for vLLM Qwen

```
genai-perf profile \
  -m Qwen/Qwen3-0.6B \
  --endpoint-type chat \
  --synthetic-input-tokens-mean 200 \
  --synthetic-input-tokens-stddev 0 \
  --output-tokens-mean 100 \
  --output-tokens-stddev 0 \
  --streaming \
  --request-count 50 \
  --warmup-request-count 10
```


Allow Firewall

sudo iptables -I INPUT -p tcp --dport 8000 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -I OUTPUT -p tcp --sport 8000 -m state --state ESTABLISHED -j ACCEPT

Docker forward too

sudo iptables -I FORWARD -p tcp --dport 8000 -j ACCEPT
sudo iptables -I FORWARD -p tcp --sport 8000 -j ACCEPT

curl -s http://<IP>:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen/Qwen3-0.6B",
    "messages": [{"role":"user","content":"Say hello in one short sentence."}],
    "temperature": 0.2
  }' | jq

Open ports for Promethues/grafana

sudo iptables -I INPUT -p tcp --dport 3000 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -I INPUT -p tcp --dport 9090 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -I OUTPUT -p tcp --sport 3000 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -I OUTPUT -p tcp --sport 9090 -m state --state ESTABLISHED -j ACCEPT
