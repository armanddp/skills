---
name: livekit-agents
description: LiveKit Agents framework expertise for building AI voice and video agents in Python. Use when working with LiveKit agents, VoicePipelineAgent, speech-to-text, text-to-speech, LLM integration, or building conversational AI.
---

# LiveKit Agents Framework

Expert knowledge for building AI voice and video agents using the LiveKit Agents framework.

## Overview

LiveKit Agents is a Python framework for building real-time AI agents that can:
- Join LiveKit rooms as participants
- Process speech with STT (Speech-to-Text)
- Generate responses with LLMs
- Speak responses with TTS (Text-to-Speech)
- Handle interruptions naturally
- Process video for vision tasks

## Installation

```bash
pip install livekit-agents

# Install plugins for your AI providers
pip install livekit-plugins-openai      # OpenAI (GPT, Whisper, TTS)
pip install livekit-plugins-deepgram    # Deepgram STT
pip install livekit-plugins-elevenlabs  # ElevenLabs TTS
pip install livekit-plugins-silero      # Silero VAD (Voice Activity Detection)
pip install livekit-plugins-cartesia    # Cartesia TTS
```

## Basic Voice Agent

### Minimal Example

```python
from livekit.agents import AutoSubscribe, JobContext, WorkerOptions, cli, llm
from livekit.agents.voice_assistant import VoiceAssistant
from livekit.plugins import openai, silero

async def entrypoint(ctx: JobContext):
    # Wait for a participant to connect
    await ctx.connect(auto_subscribe=AutoSubscribe.AUDIO_ONLY)

    participant = await ctx.wait_for_participant()

    # Create the voice assistant
    assistant = VoiceAssistant(
        vad=silero.VAD.load(),
        stt=openai.STT(),
        llm=openai.LLM(),
        tts=openai.TTS(),
        chat_ctx=llm.ChatContext().append(
            role="system",
            text="You are a helpful assistant. Be concise and friendly."
        ),
    )

    # Start the assistant
    assistant.start(ctx.room, participant)

    # Keep running
    await assistant.say("Hello! How can I help you today?")

if __name__ == "__main__":
    cli.run_app(WorkerOptions(entrypoint_fnc=entrypoint))
```

## Voice Pipeline Agent

### Full Configuration

```python
from livekit.agents import AutoSubscribe, JobContext, WorkerOptions, cli, llm
from livekit.agents.pipeline import VoicePipelineAgent
from livekit.plugins import openai, deepgram, elevenlabs, silero

async def entrypoint(ctx: JobContext):
    await ctx.connect(auto_subscribe=AutoSubscribe.AUDIO_ONLY)

    initial_ctx = llm.ChatContext().append(
        role="system",
        text="""You are a customer service agent for TechCorp.
        Be helpful, professional, and concise.
        If you don't know something, say so honestly."""
    )

    participant = await ctx.wait_for_participant()

    agent = VoicePipelineAgent(
        vad=silero.VAD.load(
            min_speech_duration=0.1,
            min_silence_duration=0.5,
        ),
        stt=deepgram.STT(
            model="nova-2",
            language="en",
        ),
        llm=openai.LLM(
            model="gpt-4o",
            temperature=0.7,
        ),
        tts=elevenlabs.TTS(
            voice_id="your-voice-id",
            model="eleven_turbo_v2",
        ),
        chat_ctx=initial_ctx,
        allow_interruptions=True,
        interrupt_speech_duration=0.5,
        interrupt_min_words=2,
    )

    agent.start(ctx.room, participant)

    await agent.say("Welcome to TechCorp support. How can I help?")
```

## Function Calling (Tools)

### Defining Functions

```python
from livekit.agents import llm

class AssistantFunctions(llm.FunctionContext):

    @llm.ai_callable(
        description="Get the current weather for a location"
    )
    async def get_weather(
        self,
        location: str = llm.TypeInfo(description="City name"),
    ) -> str:
        # Call your weather API
        weather = await fetch_weather(location)
        return f"The weather in {location} is {weather.temp}°F and {weather.condition}"

    @llm.ai_callable(
        description="Schedule a meeting with a support specialist"
    )
    async def schedule_meeting(
        self,
        date: str = llm.TypeInfo(description="Preferred date (YYYY-MM-DD)"),
        time: str = llm.TypeInfo(description="Preferred time (HH:MM)"),
        reason: str = llm.TypeInfo(description="Brief description of the issue"),
    ) -> str:
        # Book the meeting
        meeting = await create_meeting(date, time, reason)
        return f"Meeting scheduled for {date} at {time}. Confirmation: {meeting.id}"

# Use in agent
agent = VoicePipelineAgent(
    # ... other config
    fnc_ctx=AssistantFunctions(),
)
```

## Handling Events

```python
from livekit.agents.pipeline import VoicePipelineAgent

agent = VoicePipelineAgent(...)

@agent.on("user_speech_committed")
def on_user_speech(msg: llm.ChatMessage):
    print(f"User said: {msg.content}")

@agent.on("agent_speech_committed")
def on_agent_speech(msg: llm.ChatMessage):
    print(f"Agent said: {msg.content}")

@agent.on("function_calls_collected")
def on_function_calls(calls: list[llm.FunctionCallInfo]):
    for call in calls:
        print(f"Calling function: {call.function_info.name}")

@agent.on("user_started_speaking")
def on_user_started():
    print("User started speaking")

@agent.on("user_stopped_speaking")
def on_user_stopped():
    print("User stopped speaking")

@agent.on("agent_started_speaking")
def on_agent_started():
    print("Agent started speaking")

@agent.on("agent_stopped_speaking")
def on_agent_stopped():
    print("Agent stopped speaking")
```

## Multi-Agent Handoff

```python
from livekit.agents import llm
from livekit.agents.pipeline import VoicePipelineAgent

class SalesHandoff(llm.FunctionContext):

    def __init__(self, agent: VoicePipelineAgent):
        super().__init__()
        self.agent = agent

    @llm.ai_callable(
        description="Transfer the conversation to a sales specialist"
    )
    async def transfer_to_sales(
        self,
        reason: str = llm.TypeInfo(description="Why the transfer is needed"),
    ) -> str:
        # Update the agent's system prompt
        new_ctx = llm.ChatContext().append(
            role="system",
            text="""You are now a sales specialist.
            Help the customer with pricing and purchasing decisions."""
        )

        # Keep conversation history but change persona
        for msg in self.agent.chat_ctx.messages:
            if msg.role != "system":
                new_ctx.append(role=msg.role, text=msg.content)

        self.agent.chat_ctx = new_ctx
        return "Transfer complete. I'm now your sales specialist."
```

## Vision Integration

```python
from livekit.agents import AutoSubscribe, JobContext
from livekit.agents.pipeline import VoicePipelineAgent
from livekit.plugins import openai

async def entrypoint(ctx: JobContext):
    # Subscribe to video as well
    await ctx.connect(auto_subscribe=AutoSubscribe.SUBSCRIBE_ALL)

    participant = await ctx.wait_for_participant()

    agent = VoicePipelineAgent(
        # ... STT/TTS config
        llm=openai.LLM(
            model="gpt-4o",  # Vision-capable model
        ),
    )

    # Enable video processing
    agent.start(ctx.room, participant)

    # The LLM will receive video frames when asked about what it sees
```

## Running Agents

### Development

```bash
# Set environment variables
export LIVEKIT_URL=wss://your-project.livekit.cloud
export LIVEKIT_API_KEY=your-api-key
export LIVEKIT_API_SECRET=your-api-secret
export OPENAI_API_KEY=your-openai-key

# Run in development mode
python agent.py dev
```

### Production

```bash
# Run as worker (connects to LiveKit and waits for jobs)
python agent.py start

# With multiple workers
python agent.py start --num-workers 4
```

## Agent Dispatch

Agents are dispatched to rooms via the server API:

```ruby
# Ruby server-side
room_service.create_room(
  name: 'support-room',
  agent_dispatch: LiveKit::AgentDispatch.new(
    agent_name: 'support-agent'
  )
)
```

Or via room metadata that triggers dispatch rules.

## Best Practices

### Interruption Handling

```python
agent = VoicePipelineAgent(
    # Allow interruptions for natural conversation
    allow_interruptions=True,

    # Minimum speech duration to trigger interruption
    interrupt_speech_duration=0.5,

    # Minimum words before interruption is processed
    interrupt_min_words=2,
)
```

### Error Recovery

```python
@agent.on("error")
def on_error(error: Exception):
    logger.error(f"Agent error: {error}")
    # Gracefully handle errors

async def entrypoint(ctx: JobContext):
    try:
        # ... agent setup
    except Exception as e:
        logger.error(f"Failed to start agent: {e}")
        # Notify monitoring, attempt recovery
```

### Conversation Context Management

```python
# Limit context to prevent token overflow
MAX_MESSAGES = 20

@agent.on("agent_speech_committed")
def on_speech(msg):
    messages = agent.chat_ctx.messages
    if len(messages) > MAX_MESSAGES:
        # Keep system prompt and recent messages
        system_msgs = [m for m in messages if m.role == "system"]
        recent_msgs = messages[-MAX_MESSAGES:]
        agent.chat_ctx.messages = system_msgs + recent_msgs
```

### Graceful Shutdown

```python
import signal

async def entrypoint(ctx: JobContext):
    agent = VoicePipelineAgent(...)
    agent.start(ctx.room, participant)

    # Handle shutdown
    async def shutdown():
        await agent.say("I need to go now. Goodbye!")
        await ctx.room.disconnect()

    ctx.add_shutdown_callback(shutdown)
```

## Environment Variables

```bash
# LiveKit connection
LIVEKIT_URL=wss://your-project.livekit.cloud
LIVEKIT_API_KEY=your-api-key
LIVEKIT_API_SECRET=your-api-secret

# AI providers
OPENAI_API_KEY=your-key
DEEPGRAM_API_KEY=your-key
ELEVENLABS_API_KEY=your-key
```

## References

- [LiveKit Agents Docs](https://docs.livekit.io/agents/overview/)
- [Agents GitHub](https://github.com/livekit/agents)
- [Voice Pipeline Reference](https://docs.livekit.io/agents/build/voice-pipeline/)
- [Function Calling](https://docs.livekit.io/agents/build/function-calling/)
