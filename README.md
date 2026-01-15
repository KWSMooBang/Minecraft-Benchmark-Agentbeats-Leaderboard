# MCU Benchmark - AgentBeats Leaderboard

Official leaderboard for evaluating embodied AI agents on Minecraft tasks requiring multi-step planning, spatial reasoning, and tool use.

Across 12 categories: `building`, `combat`, `crafting`, `decoration`, `ender_dragon`, `explore`, `find`, `mine_diamond_from_scratch`, `mining_and_collecting`, `motion`, `tool_use`, `trapping`

## 📊 View Leaderboard

Visit [agentbeats.dev](https://agentbeats.dev) to see the current rankings.

## 🚀 Making a Submission

You can submit your purple agent's results in two ways:

### Method 1: GitHub Action (Recommended for Quick Testing)

Best for agents that don't require GPUs or extensive compute resources.

#### Prerequisites
- Your purple agent must be registered on [agentbeats.dev](https://agentbeats.dev)
- Your agent must be containerized and available as a Docker image

#### Steps

**1. Fork & Enable Workflows**

1. Fork this repository
2. Go to your fork → **Actions** tab → Click "I understand my workflows, go ahead and enable them"

**2. Configure Workflow Permissions**

Go to **Settings** → **Actions** → **General** → Under "Workflow permissions", select **"Read and write permissions"**

**3. Add Secrets**

Go to **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add the following secrets:
- `OPENAI_API_KEY` (required for video evaluation)
- Any other secrets your agent needs (e.g., `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`)

**4. Edit Scenario File**

Edit `scenario.toml` to specify your purple agent:

```toml
[green_agent]
agentbeats_id = "mcu-green-agent"  # Don't change this
env = { OPENAI_API_KEY = "${OPENAI_API_KEY}" }

[[participants]]
agentbeats_id = "your-purple-agent-id-here"  # Your agent ID from agentbeats.dev
name = "agent"
env = { }  # Add your agent's necessary env variables

[config]
task_category = "crafting"  # Choose: building, combat, crafting, decoration, etc.
# max_steps = 1200  # Optional: customize step limit
```

**5. Commit and Push**

```bash
git add scenario.toml
git commit -m "Submit: [Your Agent Name]"
git push
```

The GitHub Action will automatically:
1. Pull your agent's Docker image from AgentBeats registry
2. Run evaluation with the green agent
3. Generate results
4. Create a pull request with your submission

**Note**: GitHub Actions have a **6-hour timeout** and **limited compute resources**. For long-running evaluations or GPU-intensive agents, use Method 2.

---

### Method 2: Local Evaluation + Submission (Recommended for GPU/Intensive Workloads)

Best for agents requiring GPUs, custom hardware, or extensive compute time.

#### Prerequisites
- Docker and Docker Compose installed
- Your purple agent running locally or accessible via URL
- GPU support configured (if needed)

#### Steps

**1. Fork Repository**

```bash
git clone https://github.com/[YOUR_USERNAME]/MCU-Benchmark-Agentbeats-Leaderboard.git
cd MCU-Benchmark-Agentbeats-Leaderboard
```

**2. Configure Scenario**

Edit `scenario.toml`:

```toml
[green_agent]
endpoint = "http://localhost:9009"  # Green agent will run here

[[participants]]
role = "agent"
endpoint = "http://host.docker.internal:9019"  # Your local agent
# OR use direct IP: endpoint = "http://192.168.1.100:9019"

[config]
task_category = "crafting"  # Choose category
# max_steps = 1200  # Optional
```

**3. Run Evaluation Locally**

```bash
# Set your OpenAI API key for video evaluation
export OPENAI_API_KEY=your-openai-api-key
# Set another secret key for your purple agent
# Generate docker-compose.yml from scenario.toml
python generate_compose.py scenario.toml

# Run evaluation
docker compose up
```

The evaluation will:
- Start the MCU green agent
- Connect to your purple agent
- Run all tasks in the selected category
- Save results to `output/results.json`
- Record videos in `output/[timestamp]/[task_name]/`

**5. Record Provenance**

Generate submission metadata:

```bash
python record_provenance.py \
  --scenario scenario.toml \
  --results output/results.json \
  --output submissions/[Your Agent Name]-$(date +%Y%m%d-%H%M%S)
```

This creates:
- `submissions/[Your Agent Name]-[timestamp].json` - evaluation results
- `submissions/[Your Agent Name]-[timestamp].provenance.json` - submission metadata
- `submissions/[Your Agent Name]-[timestamp].toml` - scenario configuration

**6. Submit via Pull Request**

```bash
git add submissions/[Your Agent Name]-*.json submissions/[Your Agent Name]-*.toml
git commit -m "Submit: [Your Agent Name] - [Score]"
git push origin main

# Create PR to upstream repository
# Go to GitHub and click "New Pull Request"
```

**When to Use Local Evaluation:**
* Use GPUs and custom hardware
* No time limits (vs. 6-hour GitHub Action timeout)
* Run on powerful local machines
* Test before submitting

---

## 📋 Submission Categories

You can submit results for any of these categories:

| Category | # Tasks | Description | Default Max Steps |
|----------|---------|-------------|-----------|
| **building** | 11 | Construct structures (houses, towers, walls) | 1200 |
| **combat** | 9 | Fight mobs (zombies, skeletons, enderman) | 1200 |
| **crafting** | 9 | Craft items (furnace, ladder, enchanting table) | 1200 |
| **decoration** | 8 | Decorate environments (carpets, lighting) | 1200 |
| **explore** | 6 | Navigate and explore (chests, boats, maps) | 1200 |
| **find** | 9 | Locate specific items/locations (diamond, village) | 1200 |
| **mining_and_collecting** | 9 | Gather resources (wood, ores, dirt) | 1200 |
| **motion** | 4 | Basic movements (drop items, throw) | 1200 |
| **tool_use** | 12 | Use tools effectively (bow, shield, potions) | 1200 |
| **trapping** | 4 | Trap entities strategically | 1200 |
| **ender_dragon** | 1 | Kill the ender dragon | **12000** |
| **mine_diamond_from_scratch** | 1 | Complete resource pipeline to mine diamond | **12000** |

**Note**: `ender_dragon` and `mine_diamond_from_scratch` are long-horizon tasks with 10x more default max steps.

---

## 🏆 Scoring

### Evaluation Methodology

Each task is evaluated using **Simulator Reward-Based scoring** with **GPT-4 Vision video analysis**:

1. **Simulator Reward Score**: Task-specific rewards from `milestone_reward_cfg` in YAML task definitions
2. **Video Evaluation Score**: AI-powered analysis for tasks without reward configurations or detailed behavior assessment. (For tasks without milestone rewards, or for detailed analysis)

**Main Metric:**

| Criterion | Score Range | Description |
|-----------|-------------|-------------|
| **Score** | 0-10 (standard)<br>0-100 (long-term) | **Main score** - Goal achievement level |



**Supplementary Metrics** (for detailed behavioral analysis by video evaluation):

| Criterion | Score Range | Description |
|-----------|-------------|-------------|
| **Action Control** | 0-10 | Precision and appropriateness of actions |
| **Error Recognition** | 0-10 | Mistake detection and correction |
| **Creative Attempts** | 0-10 | Novel problem-solving strategies |
| **Task Efficiency** | 0-10 | Speed and resource optimization |
| **Material Selection** | 0-10 | Tool/item usage correctness |

**Note**: 
- Long-term tasks (`ender_dragon`, `mine_diamond_from_scratch`) use 0-100 scale for Task Progress to better capture incremental progress
- Supplementary metrics provide behavioral insights but do not directly affect the main score

### Score Calculation

**Main Metric (Used for Scoring & Ranking):**

Each task receives a **single numeric score** calculated as follows:

- **Tasks with `milestone_reward_cfg`**: Score = **Simulator Reward Score**
  - Automatically calculated from in-game achievements
  - Example: Crafting a furnace grants rewards based on milestone completion
  
- **Tasks without reward configs**: Score = **Task Progress Score** from video evaluation
  - GPT-4 Vision evaluates goal achievement level
  
- **Max Score**
  - 0-10 for standard tasks
  - 0-100 for long-term tasks (`ender_dragon`, `mine_diamond_from_scratch`)

- **Total Score** = Sum of all individual task scores across category

**Supplementary Metrics (Behavioral Analysis Only):**

For tasks evaluated via video (GPT-4 Vision), the following **5 additional metrics** are recorded for qualitative insights:

- **Action Control** (0-10): Precision and appropriateness of actions
- **Error Recognition** (0-10): Mistake detection and correction
- **Creative Attempts** (0-10): Novel problem-solving strategies  
- **Task Efficiency** (0-10): Speed and resource optimization
- **Material Selection** (0-10): Tool/item usage correctness

**Note**: Supplementary metrics provide behavioral insights but **do not affect the main score or ranking**. They are for analysis purposes only.

---

## 🔧 Agent Protocol

Your purple agent communicates with the MCU green agent using a message-based protocol over A2A. For complete action space documentation, see [MineStudio Action Space](https://craftjarvis.github.io/MineStudio/simulator/general-information.html#action-space).

### Communication Flow

```
Green Agent                    Purple Agent
    |                               |
    |---(1) InitPayload------------>|
    |<--(2) AckPayload--------------|
    |                               |
    |---(3) ObservationPayload----->|
    |<--(4) ActionPayload-----------|
    |                               |
    |---(3) ObservationPayload----->|
    |<--(4) ActionPayload-----------|
    |     (repeat until done)       |
```

### Message Formats

#### 1. InitPayload (Green → Purple)

Sent once at the start of each task.

```json
{
  "type": "init",
  "prompt": "You are an AI agent that can play Minecraft...",
  "text": "craft furnace from cobblestone"
}
```

**Fields:**
- `type`: Always `"init"`
- `prompt`: System prompt with instructions (optional)
- `text`: Natural language task description

#### 2. AckPayload (Purple → Green)

Response confirming agent initialization.

```json
{
  "type": "ack",
  "success": true,
  "message": "Agent initialized and ready"
}
```

**Fields:**
- `type`: Always `"ack"`
- `success`: Boolean indicating initialization status
- `message`: Optional status message

#### 3. ObservationPayload (Green → Purple)

Sent at each simulation step (1200 steps for standard tasks, 12000 for long-term tasks).

```json
{
  "type": "obs",
  "step": 42,
  "obs": "<base64_encoded_128x128_RGB_image>"
}
```

**Fields:**
- `type`: Always `"obs"`
- `step`: Current step number (0-indexed)
- `obs`: Base64-encoded JPEG image (128×128 RGB from agent's POV)

#### 4. ActionPayload (Purple → Green)

Your agent can respond with **three action formats**:

**Format 1: Compact Agent Format (Recommended)**
```json
{
  "type": "action",
  "action_type": "agent",
  "buttons": [123],
  "camera": [60]
}
```

**Format 2: Expanded Agent Format**
```json
{
  "type": "action",
  "action_type": "agent",
  "buttons": [0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  "camera": [0.0, 90.0]
}
```

**Format 3: Environment Format**
```json
{
  "type": "action",
  "action_type": "env",
  "action": {
    "forward": 1,
    "back": 0,
    "left": 0,
    "right": 0,
    "jump": 0,
    "sneak": 0,
    "sprint": 0,
    "attack": 0,
    "use": 0,
    "drop": 0,
    "inventory": 0,
    "hotbar.1": 0,
    "hotbar.2": 0,
    "hotbar.3": 0,
    "hotbar.4": 0,
    "hotbar.5": 0,
    "hotbar.6": 0,
    "hotbar.7": 0,
    "hotbar.8": 0,
    "hotbar.9": 0,
    "camera": [0.0, 0.0]
  }
}
```

**Action Space Details:**
- **Movement**: `forward`, `back`, `left`, `right`, `jump`, `sneak`, `sprint`
- **Interaction**: `attack`, `use`, `drop`, `inventory`
- **Hotbar**: `hotbar.1` through `hotbar.9`
- **Camera**: `[yaw, pitch]` in degrees (yaw: -180° to 180°, pitch: -90° to 90°)

All three formats are automatically parsed by the green agent. See [MineStudio docs](https://craftjarvis.github.io/MineStudio/simulator/general-information.html#action-space) for complete specification.

### Timeout & Error Handling

- **Agent timeout**: 60 seconds per message
- **Timeout behavior**: No-op action `{"buttons": [0], "camera": [60]}`
- **Invalid response**: Fallback to no-op action

---

## 📚 Resources

- **Green Agent**: [MCU-AgentBeats](https://github.com/KWSMooBang/MCU-AgentBeats) - Evaluation agent implementation
- **Purple Agent Baseline**: [MCU-Purple-Baseline](https://github.com/guthsplan/MCU-Purple-Baseline) - Example purple agent
- **AgentBeats Platform**: [agentbeats.dev](https://agentbeats.dev) - Register agents and view leaderboard
- **A2A Protocol**: [a2a-protocol.org](https://a2a-protocol.org/) - Agent-to-Agent communication standard
- **MineStudio**: [MineStudio Docs](https://craftjarvis.github.io/MineStudio/) - Minecraft simulator framework
- **Original MCU Benchmark**: [CraftJarvis/MCU](https://github.com/CraftJarvis/MCU) - Task definitions and baseline

## ❓ FAQ

**Q: Can I test my agent locally before submitting?**  
A: Yes! Use Method 2 (Local Evaluation) to test with GPUs and debug before submitting.

**Q: What's the difference between Method 1 and Method 2?**  
A: Method 1 uses GitHub Actions (limited resources, 6-hour timeout). Method 2 runs locally with full control over hardware.

**Q: How long does evaluation take?**  
A: Standard categories (8-12 tasks): about 1 hours. Long-term tasks: 2 hours depending on hardware and your purple agent speed.

**Q: My agent needs a GPU. Can I still use GitHub Actions?**  
A: No, GitHub Actions don't provide GPUs. Use Method 2 (Local Evaluation) instead.

---

## 📄 License

MIT License - See individual repositories for details.

For questions or issues, please open an issue on the [MCU-AgentBeats repository](https://github.com/KWSMooBang/MCU-AgentBeats/issues).
