## **Agent Runtime Opcode Set (v1.0)**

---

### **Control (0x00–0x07)**
```
0x00 NOP()                    No operation
0x01 HALT(exit_code)          Stop agent with exit code
0x02 JMP(addr)                Unconditional jump
0x03 JZ(addr)                 Jump if top-of-stack == 0
0x04 JNZ(addr)                Jump if top-of-stack != 0
0x05 CALL(addr)               Call subroutine, push return address
0x06 RET()                    Return from subroutine
0x07 YIELD(cycles)            Yield execution for N cycles
```

---

### **ArithConf (0x08–0x17) – Confidence Propagates**
*Confidence propagation rule:*
- For binary ops `c_result = min(c_a, c_b) * (1 - |a - b| / max(|a|,|b|,1))`
- For unary ops `c_result = c_input * stability_factor` (where stability_factor is op-specific, default 0.95)
- Confidence `c` ∈ [0,1] stored as upper bits of each 32-bit value (8-bit confidence + 24-bit signed fixed-point).

```
0x08 ADD(c)                   c = a + b, confidence propagated
0x09 SUB(c)                   c = a - b, confidence propagated
0x0A MUL(c)                   c = a * b, confidence propagated
0x0B DIV(c)                   c = a / b (b != 0), c_result = min(c_a,c_b)*0.8
0x0C MOD(c)                   c = a % b, confidence as DIV
0x0D INC(c)                   c = a + 1, c_result = c_a*0.98
0x0E DEC(c)                   c = a - 1, c_result = c_a*0.98
0x0F NEG(c)                   c = -a, c_result = c_a*0.99
0x10 ABS(c)                   c = |a|, c_result = c_a
0x11 AVG(c)                   c = (a+b)/2, c_result = sqrt(c_a*c_b)
0x12 FLOOR(c)                 c = floor(a), c_result = c_a*0.9
0x13 CEIL(c)                  c = ceil(a), c_result = c_a*0.9
0x14 ROUND(c)                 c = round(a), c_result = c_a*0.9
0x15 SQRT(c)                  c = sqrt(a), c_result = c_a*0.85
0x16 POW(c)                   c = a^b, c_result = min(c_a,c_b)*0.75
0x17 LERP(c, alpha)           c = a*(1-alpha) + b*alpha, c_result = min(c_a,c_b)
```

---

### **Logic (0x18–0x1F)**
```
0x18 AND()                    Bitwise AND
0x19 OR()                     Bitwise OR
0x1A XOR()                    Bitwise XOR
0x1B NOT()                    Bitwise NOT
0x1C SHL(count)               Shift left
0x1D SHR(count)               Shift right (logical)
0x1E ROL(count)               Rotate left
0x1F ROR(count)               Rotate right
```

---

### **Compare (0x20–0x27)**
```
0x20 CMPEQ()                  Push 1 if a == b else 0
0x21 CMPNE()                  Push 1 if a != b else 0
0x22 CMPGT()                  Push 1 if a > b else 0
0x23 CMPLT()                  Push 1 if a < b else 0
0x24 CMPGE()                  Push 1 if a >= b else 0
0x25 CMPLE()                  Push 1 if a <= b else 0
0x26 CMPZ()                   Push 1 if a == 0 else 0
0x27 CMPNZ()                  Push 1 if a != 0 else 0
```

---

### **Stack (0x28–0x2F)**
```
0x28 PUSH(val)                Push immediate value
0x29 POP()                    Remove top-of-stack
0x2A DUP()                    Duplicate top-of-stack
0x2B SWAP()                   Swap top two stack elements
0x2C OVER()                   Copy second element to top
0x2D ROT()                    Rotate third element to top
0x2E PICK(n)                  Copy nth element to top
0x2F DEPTH()                  Push current stack depth
```

---

### **Perception (0x30–0x37)**
```
0x30 SENSE(sensor_id)         Read sensor, push value+confidence
0x31 FILTER(kernel_id)        Apply filter to perception buffer
0x32 FUSE(sensor_a, sensor_b) Sensor fusion: weighted average with confidence boost
0x33 SCAN(range, angle)       Push array of detected objects in cone
0x34 TRACK(obj_id)            Push tracked object state
0x35 FORGET(obj_id)           Remove object from tracking
0x36 ATTEND(priority_mask)    Set attention focus
0x37 RESOLVE(ambiguity_id)    Resolve perceptual ambiguity via context
```

---

### **Agent-to-Agent (A2A) (0x38–0x4F)**
```
0x38 TELL(agent_id, msg)      Send message (fire-and-forget)
0x39 ASK(agent_id, query)     Send query, push response future
0x3A DELEGATE(agent_id, task) Delegate task, push promise
0x3B BROADCAST(group, msg)    Broadcast to group
0x3C LISTEN(channel)          Listen on channel, push message if any
0x3D TRUST(agent_id, delta)   Adjust trust level [-1,1]
0x3E DISTRUST(agent_id, reason) Mark agent as untrusted
0x3F VERIFY(signature)        Verify message signature
0x40 CAP_GET(agent_id)        Get capability vector of agent
0x41 CAP_CHECK(cap_mask)      Check if agent has capabilities
0x42 CAP_DELEGATE(cap_mask)   Temporarily delegate capabilities
0x43 CAP_REVOKE(cap_mask)     Revoke delegated capabilities
0x44 GROUP_JOIN(group_id)     Join agent group
0x45 GROUP_LEAVE(group_id)    Leave agent group
0x46 VOTE(proposal_id, choice) Cast vote
0x47 CONSENSUS(proposal)      Initiate consensus round
0x48 BARRIER(sync_id)         Synchronization barrier
0x49 ELECTION(role)           Initiate leader election
0x4A CONTRACT(agent_id, terms) Propose interaction contract
0x4B COMMIT(tx_id)            Commit transaction
0x4C ROLLBACK(tx_id)          Rollback transaction
0x4D NEGOTIATE(agent_id, item) Start negotiation
0x4E MEDIATE(agent_a, agent_b) Offer mediation
0x4F ARBITRATE(dispute_id)    Make binding arbitration decision
```

---

### **Memory (0x50–0x57)**
```
0x50 LOAD(addr)               Load from memory to stack
0x51 STORE(addr)              Store top-of-stack to memory
0x52 ALLOC(size)              Allocate heap memory, push pointer
0x53 FREE(ptr)                Free allocated memory
0x54 MEMCPY(dst, src, len)    Copy memory
0x55 MEMSET(ptr, value, len)  Set memory region
0x56 GC_COLLECT()             Trigger garbage collection
0x57 GC_ROOT(ptr)             Mark pointer as GC root
```

---

### **Type (0x58–0x5F)**
```
0x58 TYPEOF()                 Push type ID of value
0x59 CAST(type_id)            Cast top-of-stack to type
0x5A CHECK_TYPE(type_id)      Assert type, trap if mismatch
0x5B BOX(type_id)             Box primitive into object
0x5C UNBOX()                  Unbox object to primitive
0x5D IS_NULL()                Push 1 if null else 0
0x5E IS_REF()                 Push 1 if reference type else 0
0x5F SIZE_OF(type_id)         Push size in bytes of type
```

---

### **SIMD (0x60–0x67)**
```
0x60 VEC_ADD(vec_len)         SIMD add vectors
0x61 VEC_MUL(vec_len)         SIMD multiply vectors
0x62 VEC_DOT(vec_len)         Dot product, push scalar
0x63 VEC_NORM(vec_len)        Euclidean norm of vector
0x64 VEC_CONFUSE(vec_len)     Apply confusion matrix transform
0x65 VEC_FUSE(vec_len)        Fuse multiple sensor vectors
0x66 VEC_CLAMP(min, max)      Clamp vector elements
0x67 VEC_SELECT(mask)         Select elements by mask
```

---

### **Instinct (0x68–0x6F)**
```
0x68 ACTIVATE(pattern_addr)   Activate instinct pattern
0x69 EXPRESS(emotion_id)      Express emotion/behavior
0x6A BIND(trigger, response)  Bind stimulus to instinct response
0x6B UNBIND(binding_id)       Remove instinct binding
0x6C DRIVE(drive_id, intensity) Set drive intensity
0x6D SATISFY(drive_id, amount) Reduce drive
0x6E IMPULSE(urgency)         Execute highest-urgency impulse
0x6F INHIBIT(response_id)     Temporarily inhibit instinct response
```

---

### **Energy (0x70–0x77)**
```
0x70 ATP_REQUEST(amount)      Request ATP (energy units)
0x71 ATP_RELEASE(amount)      Release ATP back to pool
0x72 ATP_BALANCE()            Push current ATP balance
0x73 APOPTOSIS(threshold)     Initiate apoptosis if ATP < threshold
0x74 METABOLIZE(rate)         Set metabolic rate
0x75 CHARGE(capacitor_id)     Charge internal capacitor
0x76 DISCHARGE(capacitor_id)  Discharge capacitor
0x77 EFFICIENCY(factor)       Set energy efficiency factor
```

---

### **System (0x78–0x7F)**
```
0x78 TRAP(code)               Software trap/interrupt
0x79 DEBUG(breakpoint_id)     Set debug breakpoint
0x7A PROFILE(start|stop)      Start/stop performance profiling
0x7B TIME()                   Push system time in cycles
0x7C RAND(seed)               Push random number
0x7D ENCRYPT(key_ptr, len)    Encrypt memory region
0x7E DECRYPT(key_ptr, len)    Decrypt memory region
0x7F SYSINFO(info_type)       Push system information
```

---

## **Confidence Propagation in ArithConf**

Each 32-bit value in ArithConf operations is structured as:
- **Bits 31–24**: Confidence `c` (0x00–0xFF, scaled 0.0–1.0)
- **Bits 23–0**: Signed 24-bit fixed-point value (16.8 format)

**Propagation rules:**

1. **Binary operations (ADD, SUB, MUL, etc.):**
   ```
   c_result = min(c_a, c_b) * (1 - |a - b| / max(|a|, |b|, 1))
   ```
   - Higher disagreement between inputs reduces confidence
   - Minimum input confidence forms upper bound
   - Division by max(|a|,|b|,1) normalizes difference

2. **Unary operations (INC, DEC, NEG, ABS, etc.):**
   ```
   c_result = c_input * stability_factor
   ```
   - `stability_factor` is operation-specific:
     - NEG: 0.99 (negation is exact)
     - INC/DEC: 0.98 (small change, minor uncertainty)
     - SQRT: 0.85 (non-linear amplification of error)
     - POW: 0.75 (high sensitivity to input errors)

3. **Special cases:**
   - **DIV/MOD**: Additional 0.8 multiplier due to divide-by-zero risk
   - **AVG**: Geometric mean of confidences: `sqrt(c_a * c_b)`
   - **LERP**: Confidence tracks minimum input confidence

4. **Fixed-point precision:**
   - All arithmetic maintains 24-bit fixed-point precision
   - Confidence bits remain separate from value bits
   - Rounding/overflow may reduce confidence by fixed factors

**Example:**
```
ADD with:
  a = 0x800000C0 (c=0.5, value=3.0)
  b = 0x40000080 (c=0.25, value=1.5)
  
  min_confidence = 0.25
  difference = |3.0 - 1.5| = 1.5
  max_magnitude = max(3.0, 1.5, 1) = 3.0
  agreement = 1 - 1.5/3.0 = 0.5
  
  c_result = 0.25 * 0.5 = 0.125
  value_result = 4.5
  
  Result: 0x20000120 (c=0.125, value=4.5)
```

This system enables agents to maintain uncertainty estimates through computational chains, supporting probabilistic reasoning and graceful degradation with uncertain data.