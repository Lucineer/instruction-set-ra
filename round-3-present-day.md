# 2026 Agent Instruction Set Architecture (AISA)

## 1. Variable-Length Bytecode Format

### Header Structure (8 bytes)
```
[VERSION:1][FLAGS:1][SEG_COUNT:1][RESERVED:1][TOTAL_SIZE:4]
```
- **VERSION**: 0x01 for AISA v1
- **FLAGS**: Bit 0=Trusted, Bit 1=Encrypted, Bit 2=Compressed
- **SEG_COUNT**: Number of segments (1-4)
- **TOTAL_SIZE**: Total bytecode size in bytes (including header)

### Segment Descriptor (per segment, 8 bytes)
```
[TYPE:1][ATTR:1][OFFSET:2][SIZE:4]
```
- **TYPE**: 0=Code, 1=Data, 2=Capabilities, 3=Messages
- **ATTR**: RWX permissions (bits 0-2), Compression (bits 3-4)

### Instruction Encoding
```
Base Format: [OPCODE:1][MODES:1][OPERANDS...]
```

**Operand Types:**
- **Immediate**: `[TYPE:1][SIZE:1][DATA...]` (TYPE: 0=int, 1=float, 2=string)
- **Register**: `[REG:1]` (0-15)
- **Memory**: `[BASE_REG:1][OFFSET:2]`
- **Capability**: `[CAP_ID:2][PERMS:1]`

## 2. Core Registers

| Register | Name     | Purpose                              | Width  |
|----------|----------|--------------------------------------|--------|
| R0       | CONF     | Confidence (0.0-1.0)                | 32-bit float |
| R1       | INTENT   | Current intent vector               | 128-bit |
| R2       | ENERGY   | Computational budget                | 64-bit int |
| R3       | TRUST    | Trust score (0.0-1.0)               | 32-bit float |
| R4-R7    | TEMP     | Temporary computation               | 64-bit |
| R8-R11   | DATA     | Data pointers                       | 64-bit |
| R12-R15  | CTRL     | Control/status                      | 64-bit |

## 3. Confidence Propagation Formula

```
post_conf = 1 / (1/c1 + 1/c2 - 1/(c1*c2))
```
Where c1, c2 ∈ [0,1]

**Bytecode implementation:**
```assembly
; Input: R0 = c1, R1 = c2 (as float in low 32 bits)
; Output: R0 = post_conf

CONF_FUSE:
    MOVF R4, R0           ; c1
    MOVF R5, R1           ; c2
    
    ; 1/c1
    MOVF R6, #1.0
    FDIV R6, R6, R4       ; R6 = 1/c1
    
    ; 1/c2
    MOVF R7, #1.0
    FDIV R7, R7, R5       ; R7 = 1/c2
    
    ; 1/(c1*c2)
    FMUL R8, R4, R5       ; c1*c2
    MOVF R9, #1.0
    FDIV R9, R9, R8       ; R9 = 1/(c1*c2)
    
    ; 1/c1 + 1/c2 - 1/(c1*c2)
    FADD R10, R6, R7
    FSUB R10, R10, R9
    
    ; 1 / result
    MOVF R11, #1.0
    FDIV R0, R11, R10
    RET
```

## 4. Memory Architecture

### Stack (per-thread, 64KB)
- Grows downward from stack pointer (SP=R13)
- Frame pointer (FP=R14) for local variables

### Heap (shared, dynamic)
- Managed by capability system
- Allocations via `HEAP_ALLOC`/`HEAP_FREE`

### Capability Regions
```
Capability Table (per agent):
[ID:2][BASE:8][LIMIT:8][PERMS:1][TYPE:1]
```
- **PERMS**: R=0x1, W=0x2, X=0x4, D=0x8 (delegate)
- **TYPE**: 0=Memory, 1=Device, 2=Agent, 3=Service

## 5. A2A (Agent-to-Agent) Message Encoding

### Message Header (16 bytes)
```
[VERSION:1][TYPE:1][PRIO:1][TTL:1][SENDER_ID:8][SEQ:4]
```
- **TYPE**: 0=Data, 1=Query, 2=Response, 3=Intent, 4=TrustUpdate
- **PRIO**: 0-255 priority
- **TTL**: Time-to-live in cycles

### Message Body Format
```
[INTENT_VEC:16][CONFIDENCE:4][TRUST:4][ENERGY_BUDGET:8]
[PAYLOAD_TYPE:1][PAYLOAD_SIZE:2][PAYLOAD...]
[SIGNATURE:32]  ; Optional, based on flags
```

## Unified Opcode Set (135 total)

### Category 1: Core Computation (25 ops)
```
0x00-0x18: ADD, SUB, MUL, DIV, MOD
0x19-0x1F: AND, OR, XOR, NOT, SHL, SHR
0x20-0x24: FADD, FSUB, FMUL, FDIV, FSQRT
```

### Category 2: Control Flow (20 ops)
```
0x25-0x2F: JMP, JZ, JNZ, CALL, RET
0x30-0x34: LOOP, BREAK, SWITCH, YIELD, HALT
```

### Category 3: Memory (30 ops)
```
0x35-0x44: LOAD, STORE, MOV, PUSH, POP
0x45-0x4E: HEAP_ALLOC, HEAP_FREE, MEMCPY, MEMSET
```

### Category 4: CUDA Integration (35 ops)
```
0x4F-0x63: CUDA_LAUNCH, CUDA_SYNC, STREAM_CREATE
0x64-0x71: TENSOR_ALLOC, TENSOR_LOAD, MATMUL, CONV
```

### Category 5: Agent Intelligence (25 ops)
```
0x72-0x8A: CONF_FUSE, TRUST_UPDATE, INTENT_MERGE
0x8B-0x8F: ENERGY_CHECK, CAP_CHECK, MSG_SEND, MSG_RECV
```

## Byte Examples

### Example 1: Simple confidence fusion
```python
# Bytecode for CONF_FUSE operation
bytecode = bytes([
    0x01, 0x00, 0x01, 0x00,  # Header: v1, 1 segment, 32 total
    0x00, 0x07, 0x00, 0x08, 0x00, 0x00, 0x00, 0x20,  # Code segment
    
    # CONF_FUSE instruction
    0x72,                    # OPCODE: CONF_FUSE
    0x00,                    # MODES: register only
    0x00,                    # R0 = c1
    0x01,                    # R1 = c2
    # Result in R0
])
```

### Example 2: A2A Message Send
```python
# Send trust update to another agent
msg = bytes([
    # Message Header (16 bytes)
    0x01,                    # Version 1
    0x04,                    # Type: TrustUpdate
    0x80,                    # Priority 128, encrypted
    0x64,                    # TTL 100 cycles
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xAB, 0xCD,  # Sender ID
    0x00, 0x00, 0x00, 0x01,  # Sequence 1
    
    # Message Body
    # Intent vector (16 bytes) - all zeros for this example
    *([0] * 16),
    
    # Confidence (4 bytes as float32)
    0x3F, 0x80, 0x00, 0x00,  # 1.0
    
    # Trust (4 bytes as float32)
    0x3F, 0x00, 0x00, 0x00,  # 0.5
    
    # Energy budget (8 bytes as int64)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x01, 0x00, 0x00,  # 65536
    
    # Payload
    0x01,                    # Payload type: float array
    0x00, 0x08,              # 8 bytes payload
    0x3F, 0x80, 0x00, 0x00,  # 1.0
    0x3F, 0x00, 0x00, 0x00,  # 0.5
])

# Instruction to send message
send_instr = bytes([
    0x8E,                    # OPCODE: MSG_SEND
    0x01,                    # MODES: immediate target
    0x02,                    # Operand type: capability
    0x00, 0x03,              # Capability ID 3
    0x07,                    # RWD permissions
    0x01,                    # Operand type: immediate
    0x00,                    # Data type: raw bytes
    0x00, 0x40,              # 64 bytes
    *msg[:64]                # First 64 bytes of message
])
```

### Example 3: CUDA Tensor Operation
```python
# Matrix multiplication with confidence weighting
cuda_code = bytes([
    0x65,                    # OPCODE: TENSOR_ALLOC
    0x03,                    # MODES: 3 operands
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x08, 0x00,  # 2048 elements
    0x01, 0x02,              # FLOAT32 type
    0x08,                    # In R8
    
    0x66,                    # OPCODE: TENSOR_LOAD
    0x02,                    # MODES: 2 operands
    0x08,                    # Destination tensor in R8
    0x01,                    # Source: immediate data
    0x00, 0x00, 0x20, 0x00,  # 8192 bytes
    
    0x6A,                    # OPCODE: MATMUL
    0x13,                    # MODES: with confidence
    0x08, 0x09, 0x0A,        # A, B, C tensors
    0x00,                    # Use R0 (CONF) for weighting
])
```

## Execution Model

1. **Fetch**: Read instruction from code segment
2. **Decode**: Parse variable-length operands
3. **Check**: Validate capabilities and energy budget
4. **Execute**: Run operation, update confidence/trust
5. **Commit**: Write results, update energy counter

This design unifies:
- **FLUX VM's** security and isolation model
- **CUDA-Genepool's** evolutionary optimization instincts
- **CUDA-Axiom's** tensor computation primitives
- **Agent-specific** confidence, trust, and intent propagation