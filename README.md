# llm-evaluation-with-lumigator

[Lumigator](https://mozilla-ai.github.io/lumigator/) is a open-source framework work supported by Mozilla to evaluate LLM use cases. It supports teams to evaluate which LLM is best for their use case. Below are steps to set it up and work through it.

References:
1. [Website](https://www.mozilla.ai/lumigator)
2. [Documentation](https://mozilla-ai.github.io/lumigator/index.html)

Sample Notebook Walkthrough: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mozilla-ai/lumigator/blob/main/notebooks/walkthrough.ipynb)

```bash
git clone https://github.com/mozilla-ai/lumigator
cd lumigator
```

```bash
# Make sure to run below command if your docker version does not support docker compose command
# and instead you have docker-compose command to run docker-compose files.
sed -i 's/docker compose/docker-compose/g' Makefile
```

```bash
make start-lumigator
```

```bash
# Download dataset
wget https://raw.githubusercontent.com/mozilla-ai/lumigator/0bef1965c5180f39832e2932b59ef797b0853ff4/\
lumigator/python/mzai/sample_data/dialogsum_exc.csv
```

```bash
curl -s http://localhost:8000/api/v1/datasets/ \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'dataset=@'"dialogsum_exc.csv"';type=text/csv' \
  -F 'format=experiment' | jq
```

```bash
export EVAL_NAME="test_run_hugging_face" \
       EVAL_DESC="Test run for Huggingface model" \
       EVAL_MODEL="hf://Falconsai/text_summarization" \
       EVAL_DATASET="$(curl -s http://localhost:8000/api/v1/datasets/ | jq -r '.items | .[0].id')" \
       EVAL_MAX_SAMPLES="10"

export JSON_STRING=$(jq -n \
        --arg name "$EVAL_NAME" \
        --arg desc "$EVAL_DESC" \
        --arg model "$EVAL_MODEL" \
        --arg dataset_id "$EVAL_DATASET" \
        --arg max_samples "$EVAL_MAX_SAMPLES" \
        '{name: $name, description: $desc, model: $model, dataset: $dataset_id, max_samples: $max_samples}')

# Run an evaluation job.
curl -s http://localhost:8000/api/v1/jobs/evaluate/ \
     -H 'Accept: application/json' \
     -H 'Content-Type: application/json' \
     -d "$JSON_STRING" | jq
```

```bash
# Check status of evaluation job.
export SUBMISSION_ID=$(curl -s http://localhost:8000/api/v1/health/jobs/ | jq -r 'sort_by(.start_time) | reverse | .[0] | .submission_id')
curl -s "http://localhost:8000/api/v1/health/jobs/$SUBMISSION_ID" \
     -H 'Accept: application/json' | jq
```

```bash
# Check results of the submission job.
curl -s http://localhost:8000/api/v1/jobs/$SUBMISSION_ID/result/download   -H 'accept: application/json' | jq
```
