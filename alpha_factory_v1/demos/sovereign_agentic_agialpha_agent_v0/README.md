# Sovereign Agentic AGI Alpha Agent Demo

A minimal showcase of a self-directed agent with token-gated access.
Run `./deploy_sovereign_agentic_agialpha_agent_v0.sh` to build and launch the containerized environment.

## Disclaimer
References to AGI or superintelligence describe the aspirational design goals of this demo. The agent shown here is not a true artificial general intelligence and should not be treated as such.

## Features
1. Docker-based deployment with one command.
2. Flask web interface served at `http://localhost:5000`.
3. Phantom wallet gating requiring a configurable token balance.
4. Integrated agent chat backed by a language model.
5. Agentic tree search explorer for open-ended strategy discovery.
6. Built-in arithmetic evaluator for the `Calculate` tool (supports +, -, *, /, and power).

```bash
./deploy_sovereign_agentic_agialpha_agent_v0.sh
```

## Usage Tips
- Ensure Docker and docker-compose are installed.
- The script will guide you through optional model configuration.
- Press `Ctrl+C` to stop logs after deployment if desired.
