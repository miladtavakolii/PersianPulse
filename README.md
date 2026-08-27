# PersianPulse 📰🤖

**PersianPulse** is a scalable, LLM-powered pipeline for continuously crawling, preprocessing, and analyzing the sentiment of Persian news articles.

The system is designed as a modular asynchronous pipeline where newly published Persian news can be collected, cleaned, processed, and passed to an LLM-based sentiment engine with minimal manual intervention.

---

## ✨ Overview

Traditional sentiment-analysis systems often rely on a fixed classification model trained on a predefined dataset.

This project takes a different approach by combining:

* 🕷️ **Real-time news crawling**
* 🧹 **Persian text preprocessing**
* 🧠 **LLM-based sentiment analysis**
* 📨 **Asynchronous message processing**
* ⏰ **Scheduled crawling**
* 🔌 **Multiple LLM backends**
* 📊 **Data visualization**
* ⚙️ **Modular configuration**

The overall architecture is:

```text
                     ┌─────────────────────┐
                     │   News Websites     │
                     └──────────┬──────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │   Scrapy Crawler    │
                     └──────────┬──────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │     RabbitMQ        │
                     │   Message Queue     │
                     └──────────┬──────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │ Preprocess Worker   │
                     │      + Hazm         │
                     └──────────┬──────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │     RabbitMQ        │
                     └──────────┬──────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │ Sentiment Worker    │
                     │                     │
                     │  Gemini / Ollama    │
                     └──────────┬──────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │ Processed Results   │
                     └──────────┬──────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │ Dashboard / Data    │
                     │    Visualization    │
                     └─────────────────────┘
```

---

## 🎯 Project Goals

The main goals of this project are:

1. Continuously collect newly published Persian news.
2. Normalize and clean Persian text automatically.
3. Decouple data collection from NLP processing.
4. Perform sentiment analysis using Large Language Models.
5. Support both cloud-based and local LLM inference.
6. Make the pipeline scalable through asynchronous workers.
7. Provide a foundation for monitoring sentiment trends in Persian news.

---

## 🧠 Why LLM-Based Sentiment Analysis?

Persian news can contain complex linguistic structures, implicit opinions, political context, sarcasm, and domain-specific terminology.

A simple keyword-based system or traditional classifier may struggle with these cases.

By using an LLM, the system can reason over the broader context of an article instead of relying only on individual sentiment-bearing words.

The architecture also abstracts the LLM behind a common sentiment engine interface, allowing different providers to be used without changing the rest of the pipeline.

Currently supported backends include:

* **Google Gemini**
* **Ollama**

The sentiment engine is implemented as a modular component under:

```text
sentiment_engine/
├── base.py
├── engine.py
├── gemini_client.py
└── ollama_client.py
```

---

## 🏗️ Architecture

The project follows a producer/consumer architecture.

### 1. News Crawling

The crawling layer is implemented using **Scrapy**.

```text
scrapy_app/
├── spiders/
├── items.py
├── middlewares.py
├── pipelines.py
└── settings.py
```

The crawler is responsible for collecting news data from configured sources and passing the resulting items into the processing pipeline.

---

### 2. Scheduling

News crawling can be executed periodically using the scheduler component.

```text
scheduler/
├── scrapy_scheduler.py
└── write_last_timestamp.py
```

This allows the system to maintain a continuous ingestion workflow instead of requiring the crawler to be started manually for every run.

---

### 3. Message Queue

The pipeline uses **RabbitMQ** to decouple different stages of processing.

```text
Crawler
   │
   ▼
RabbitMQ
   │
   ▼
Preprocessing Worker
   │
   ▼
RabbitMQ
   │
   ▼
Sentiment Worker
```

This architecture provides several advantages:

* Independent processing stages
* Better fault isolation
* Asynchronous execution
* Easier horizontal scaling
* Back-pressure handling
* Ability to add additional consumers

RabbitMQ is configured through Docker Compose and exposes both its messaging port and management dashboard.

---

## 🧹 Persian Text Preprocessing

Persian text requires normalization before being passed to an NLP model.

The project includes a dedicated preprocessing module:

```text
preprocessing/
└── clean_text.py
```

The preprocessing stage is based on **Hazm** and is isolated from the crawler and sentiment engine.

This separation makes it possible to modify the text-cleaning pipeline without changing the crawling or inference components.

---

## 🤖 Sentiment Engine

The sentiment engine provides an abstraction over different LLM providers.

```text
sentiment_engine/
│
├── base.py
├── engine.py
├── gemini_client.py
└── ollama_client.py
```

The architecture makes it possible to switch between:

```text
                ┌───────────────┐
                │ Sentiment     │
                │ Engine        │
                └───────┬───────┘
                        │
               ┌────────┴────────┐
               │                 │
               ▼                 ▼
          ┌─────────┐       ┌─────────┐
          │ Gemini  │       │ Ollama  │
          └─────────┘       └─────────┘
```

### Google Gemini

Useful when cloud-based inference is preferred.

### Ollama

Useful when the model should run locally without sending article content to an external API.

This makes the project suitable for experimenting with both hosted and self-hosted LLMs.

---

## ⚙️ Workers

The processing pipeline contains dedicated workers:

```text
workers/
├── preprocess_worker.py
└── sentiment_worker.py
```

### Preprocess Worker

Consumes crawled articles and performs text preprocessing before forwarding them to the next stage.

### Sentiment Worker

Consumes preprocessed articles and sends them to the configured LLM sentiment backend.

Separating these responsibilities allows each stage to scale independently.

For example:

```text
        1 Crawler
            │
            ▼
       ┌─────────┐
       │RabbitMQ │
       └────┬────┘
            │
       ┌────┴────┐
       ▼         ▼
   Worker 1   Worker 2
       │         │
       └────┬────┘
            ▼
       Sentiment Queue
            │
       ┌────┴────┐
       ▼         ▼
   Worker 1   Worker 2
```

---

## 📊 Data & Visualization

The project includes dependencies for data processing and visualization using:

* **Pandas**
* **Plotly**
* **Streamlit**

This provides a foundation for building dashboards around the collected news and sentiment results.

Potential visualizations include:

* Sentiment distribution
* Sentiment over time
* News volume over time
* Sentiment by news source
* Sentiment by category
* Most frequent topics
* Positive/negative trend changes

---

## 📁 Project Structure

```text
RealTime-PersianNews-Sentiment-LLM/
│
├── config/
│   ├── prompt_template.txt
│   └── settings.yaml
│
├── data/
│   └── ...
│
├── preprocessing/
│   ├── __init__.py
│   └── clean_text.py
│
├── scheduler/
│   ├── __init__.py
│   ├── scrapy_scheduler.py
│   └── write_last_timestamp.py
│
├── scrapy_app/
│   ├── spiders/
│   ├── __init__.py
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   └── settings.py
│
├── sentiment_engine/
│   ├── __init__.py
│   ├── base.py
│   ├── engine.py
│   ├── gemini_client.py
│   └── ollama_client.py
│
├── utils/
│   ├── __init__.py
│   ├── config_manager.py
│   ├── date_parser.py
│   ├── logger.py
│   ├── rabbitmq.py
│   └── sanitize_filename.py
│
├── workers/
│   ├── __init__.py
│   ├── preprocess_worker.py
│   └── sentiment_worker.py
│
├── .env.example
├── docker-compose.yml
├── pyproject.toml
├── scrapy.cfg
└── uv.lock
```

The repository currently separates configuration, crawling, preprocessing, scheduling, sentiment inference, workers, and utilities into independent modules.

---

## 🛠️ Tech Stack

| Component          | Technology             |
| ------------------ | ---------------------- |
| Language           | Python 3.11+           |
| Web Crawling       | Scrapy                 |
| Persian NLP        | Hazm                   |
| LLM                | Google Gemini / Ollama |
| Message Queue      | RabbitMQ               |
| Scheduling         | APScheduler            |
| Data Processing    | Pandas                 |
| Visualization      | Plotly                 |
| Dashboard          | Streamlit              |
| Configuration      | YAML + `.env`          |
| Containerization   | Docker Compose         |
| Package Management | uv                     |

The project currently declares Python `>=3.11` and includes dependencies such as Scrapy, Hazm, Google GenAI, Ollama, APScheduler, Pika, Pandas, Plotly, Streamlit, PyYAML, and python-dotenv.

---

## 🚀 Installation

### Requirements

* Python `3.11+`
* `uv`
* Docker
* Docker Compose
* An LLM provider:

  * Google Gemini API, or
  * Local Ollama installation

---

### 1. Clone the Repository

```bash
git clone https://github.com/miladtavakolii/RealTime-PersianNews-Sentiment-LLM.git

cd RealTime-PersianNews-Sentiment-LLM
```

---

### 2. Install Dependencies

Using `uv`:

```bash
uv sync
```

Activate the virtual environment:

```bash
source .venv/bin/activate
```

The project is configured for Python 3.11 or newer.

---

## 🔐 Configuration

Copy the example environment file:

```bash
cp .env.example .env
```

Configure the required variables according to the selected LLM provider and RabbitMQ setup.

A typical configuration may include:

```env
GEMINI_API_KEY=your_api_key

RABBITMQ_USER=your_user
RABBITMQ_PASS=your_password
```

> Never commit `.env` or API keys to the repository.

The project already includes `.env.example` and uses `python-dotenv` for environment-based configuration.

---

## 🐇 Start RabbitMQ

RabbitMQ is provided through Docker Compose.

Start the service:

```bash
docker compose up -d
```

The included Compose configuration exposes:

```text
5672   → RabbitMQ messaging
15672  → RabbitMQ management UI
```

The management dashboard can then be accessed through:

```text
http://localhost:15672
```

---

## 🕷️ Running the Crawler

The crawler is implemented as a Scrapy application.

The project contains:

```text
scrapy_app/
├── spiders/
├── items.py
├── pipelines.py
└── settings.py
```

Run the crawler using the appropriate Scrapy command configured for the project:

```bash
scrapy list
```

Then execute the desired spider:

```bash
scrapy crawl <spider_name>
```

---

## ⚙️ Running Workers

The pipeline requires workers to consume messages from RabbitMQ.

Start the preprocessing worker:

```bash
python -m workers.preprocess_worker
```

Start the sentiment worker:

```bash
python -m workers.sentiment_worker
```

The workers are intentionally separated so they can be started, monitored, and scaled independently.

---

## ⏰ Scheduled Crawling

The scheduler component is responsible for periodically triggering the crawling process.

The relevant implementation is located in:

```text
scheduler/scrapy_scheduler.py
```

This enables a workflow such as:

```text
Every N minutes
      │
      ▼
Start crawler
      │
      ▼
Collect new articles
      │
      ▼
Publish to RabbitMQ
      │
      ▼
Process asynchronously
```

---

## 🧠 Prompt Configuration

LLM instructions are separated from the Python implementation.

The main prompt template is located at:

```text
config/prompt_template.txt
```

This makes it possible to modify the sentiment-analysis behavior without modifying the sentiment engine itself.

Other application settings are stored in:

```text
config/settings.yaml
```

This separation keeps application configuration and model instructions independent from the implementation.

---

## 🔄 End-to-End Pipeline

A typical execution flow looks like this:

```text
┌───────────────────┐
│ Persian News      │
│ Websites          │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Scrapy Spider     │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ RabbitMQ          │
│ Queue             │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Preprocess Worker │
│                   │
│ Hazm              │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ RabbitMQ          │
│ Sentiment Queue   │
└─────────┬─────────┘
          │
          ▼
┌──────────────────────────┐
│ Sentiment Worker         │
│                          │
│ Gemini / Ollama          │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ Sentiment Results        │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ Data / Dashboard         │
│                          │
│ Pandas + Plotly +        │
│ Streamlit                │
└──────────────────────────┘
```

---

## 📈 Scalability

One of the main design goals is to avoid coupling crawling speed with LLM inference speed.

For example, if the crawler collects 100 articles while the LLM can only process 20 articles per minute, RabbitMQ can temporarily buffer the workload.

Additional workers can then be started:

```text
                RabbitMQ
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
    Worker 1    Worker 2    Worker 3
       │           │           │
       └───────────┼───────────┘
                   ▼
              LLM Backend
```

This allows the system to scale processing independently from data collection.

---

## 🔌 LLM Backend Abstraction

The sentiment engine is designed around a provider abstraction.

This allows the system to use:

```text
                 SentimentEngine
                       │
          ┌────────────┴────────────┐
          │                         │
          ▼                         ▼
       Gemini                    Ollama
      Cloud API                Local Model
```

This is useful for comparing:

* Cloud vs. local inference
* Different LLM models
* Inference cost
* Latency
* Output quality
* Privacy characteristics

---

## 🧪 Experimentation

Because the LLM provider is separated from the rest of the pipeline, the project can be used to experiment with different models and prompts.

Potential experiments include:

* Different LLM architectures
* Different Persian prompts
* Few-shot vs. zero-shot classification
* Different sentiment label schemes
* Prompt-based explanation generation
* Local vs. cloud models
* Batch vs. real-time inference

---

## ⚠️ Limitations

LLM-based sentiment analysis has several limitations:

* LLM outputs may not always be deterministic.
* Sentiment can be ambiguous in news articles.
* Political and domain-specific language may require additional prompt engineering.
* Different models can produce different classifications.
* API-based inference introduces latency and cost.
* Local models depend heavily on available hardware.

For production applications, model outputs should therefore be evaluated against a manually labeled Persian news dataset.

---

## 🔒 Privacy

When using a cloud LLM such as Gemini, article content is sent to the corresponding external API.

If data privacy is important, the project can instead be configured to use a locally hosted model through Ollama.

Always review the terms and data-handling policies of the selected model provider before processing sensitive information.

---

## 🤝 Contributing

Contributions, experiments, and research ideas are welcome.

If you find a bug or have an idea for improving the training pipeline, feel free to open an issue or submit a pull request.

---

## 📄 License

License information will be added to the repository.

---

## 👤 Author

**Milad Tavakoli**

GitHub: [@miladtavakolii](https://github.com/miladtavakolii)

---

## ⭐ Acknowledgements

This project builds upon the following open-source technologies:

* Scrapy
* Hazm
* RabbitMQ
* APScheduler
* Google Gemini
* Ollama
* Pandas
* Plotly
* Streamlit
* Python

---

## 📌 Project Status

This is an **experimental research and engineering project**.

The repository currently provides the core components for:

* Real-time crawling
* Persian text preprocessing
* Asynchronous processing
* RabbitMQ-based messaging
* Scheduled ingestion
* LLM-powered sentiment analysis
* Gemini and Ollama backends
* Data processing and visualization infrastructure

The architecture is intended to serve as a foundation for further experimentation with real-time Persian NLP and LLM-based news intelligence.
