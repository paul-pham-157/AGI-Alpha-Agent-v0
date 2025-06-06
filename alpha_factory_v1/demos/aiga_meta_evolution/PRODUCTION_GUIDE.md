# 🛠️ Production Deployment Guide — AI‑GA Meta‑Evolution

This short guide distils the steps required to run the **AI‑GA Meta‑Evolution** service in a production or workshop environment.

1. **Prepare the configuration**
   - Copy `config.env.sample` to `config.env` and edit as needed.
   - Set `OPENAI_API_KEY` to enable cloud models. Leave empty to run fully offline via the bundled Mixtral model.
   - Optionally enable the Google ADK gateway by setting `ALPHA_FACTORY_ENABLE_ADK=true`. A token can be enforced with `ALPHA_FACTORY_ADK_TOKEN`.
   - For API protection, set `AUTH_BEARER_TOKEN` or provide `JWT_PUBLIC_KEY`/`JWT_ISSUER` values.
   - Verify all Python packages are available:
     Run the following command from the project root directory:
     ```bash
     AUTO_INSTALL_MISSING=1 python alpha_factory_v1/check_env.py
     ```
     This attempts to install `openai-agents`, `google-adk` and other required
     packages if they are missing. Offline environments can point the script to
     a wheelhouse via `WHEELHOUSE=/path/to/wheels`.

2. **Launch the service**
   - Using Docker:
     ```bash
     ./run_aiga_demo.sh --pull           # add --gpu for NVIDIA runtime
     ```
   - Or natively via Python:
     ```bash
     pip install -r ../../requirements.txt
     python agent_aiga_entrypoint.py
     ```

3. **Run in Colab**
   - Open the notebook at
     [colab_aiga_meta_evolution.ipynb](colab_aiga_meta_evolution.ipynb).
     Click the “Open in Colab” badge and run the setup cell. The notebook
     launches the same service with a public Gradio URL and includes test
     and API usage examples.

4. **Access the interface**
   - Gradio dashboard: [http://localhost:7862](http://localhost:7862)
   - OpenAPI docs: [http://localhost:8000/docs](http://localhost:8000/docs)
   - Prometheus metrics: [http://localhost:8000/metrics](http://localhost:8000/metrics)

5. **Persisting state**
   - Checkpoints are written to the directory specified by `CHECKPOINT_DIR` (default `./checkpoints`).
   - The service automatically reloads the latest checkpoint on start-up.

6. **Shutting down**
   - Docker: `./run_aiga_demo.sh --stop`
   - Python: send `SIGINT` or `SIGTERM`; the service will persist state before exiting.

For troubleshooting or advanced options see `README.md` in the same directory.

