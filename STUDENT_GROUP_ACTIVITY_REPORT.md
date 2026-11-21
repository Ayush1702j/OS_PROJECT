# STUDENT GROUP ACTIVITY REPORT
**Operating Systems – Activity 2 | AI-Assisted Dashboard Development and Deadlock Analysis**

---

## 1. Group Information

| Division / Group No. | Dashboard / Project Title |
| :--- | :--- |
| **B.Tech CSE / Group Activity** | **Deadlock Analysis & Resource Management System** |

| Sr. No. | PRN / Roll No. | Student Name |
| :---: | :---: | :--- |
| **1** | 202401040128 / Roll No. 32 | **Ayush Jugseniye**  |
| **2** | 202401040124 / Roll No. 29 | **Tejas Jaju** |
| **3** | 202401040272 / Roll No. 70 | **Haider Shikalgar** |

---

## 2. Project Links

| Required Submission | Link / File Name |
| :--- | :--- |
| **Working dashboard** | Local Streamlit Suite (`http://localhost:8501`) via `python -m streamlit run app.py` |
| **Complete source code** | **GitHub Repository**: [https://github.com/Ayush1702j/OS_PROJECT.git](https://github.com/Ayush1702j/OS_PROJECT.git) |

### Brief Description of Dashboard Functionality:
The project is a comprehensive Operating Systems simulation suite implementing all four classical deadlock handling strategies: Avoidance, Prevention, Detection, and Recovery. It features an interactive **Banker’s Algorithm Engine** with automatic Need matrix calculation, step-by-step Work vector execution tracing, and strict 4-state request dispatching (Granted, Waiting/Deferred, Denied/Rolled Back, and Invalid Input). Additionally, it integrates a **Shoshani-Coffman Multi-Instance Detection Routine**, an interactive vector **Resource Allocation Graph (RAG)**, victim process termination recovery, and a **Deadlock Prevention Sandbox** demonstrating Coffman condition invalidation.

---

## 3. Test-Case Analysis

| ID | Test Scenario | Expected Result | Actual Result / Evidence | Analysis / Conclusion |
| :---: | :--- | :--- | :--- | :--- |
| **TC1** | **Safe State** | Safe sequence found | Safe sequence generated: $\langle P_1 \to P_3 \to P_4 \to P_0 \to P_2 \rangle$ | Every process satisfies $Need \le Work$ sequentially; system remains guaranteed deadlock-free. |
| **TC2** | **Request Exceeds Available** | Not granted immediately; allocation unchanged | Status: `WAITING / DEFERRED`. Matrices strictly unchanged. | $P_0$ requests $[0, 4, 1] \le Need[0]$, but $4 > Available_B (3)$. Process is placed in wait queue without altering system state. |
| **TC3** | **Request Makes State Unsafe** | Request denied; original state retained | Status: `DENIED & ROLLED BACK`. Original state restored. | $P_4$ requests $[3, 3, 0] \le Available$, leaving $Avail = [0, 0, 2]$. Safety check fails; Banker's algorithm prevents unsafe state entry. |
| **TC4** | **Deadlock Detection & Recovery** | Deadlock identified by selected method | Deadlock confirmed among $\{P_0, P_1, P_2\}$. Terminated $P_2$, freeing $[1, 1]$ to resolve deadlock. | Shoshani-Coffman detection algorithm flags circular wait. Process termination frees locks, enabling remaining threads to finish. |
| **TC5** | **Deadlock Prevention** | Selected necessary condition eliminated | Backward request rejected: $Rank(R_{req}) \le Rank(R_{held})$ | Imposing linear ordering protocol ($F(R_0) < F(R_1) < F(R_2)$) eliminates circular wait condition mathematically. |
| **TC6** | **Boundary / Invalid Inputs** | Correct processing / validation message | Exception caught: `"Negative resource quantities are prohibited"` & Max Need violations | Rigorous input sanitization prevents buffer/state corruption and separates runtime states from errors. |

---

### Detailed Test-Case Technical Breakdowns

#### TC1: Need Matrix, Work Updates, and Safe Sequence
- **System Configuration**: 5 Processes ($P_0$–$P_4$), 3 Resources ($A=10, B=5, C=7$).
  - $\text{Available} = [3, 3, 2]$
  - $\text{Allocation} = [[0,1,0], [2,0,0], [3,0,2], [2,1,1], [0,0,2]]$
  - $\text{Maximum} = [[7,5,3], [3,2,2], [9,0,2], [2,2,2], [4,3,3]]$
- **Calculated Need Matrix ($\text{Need} = \text{Max} - \text{Alloc}$)**:
  - $P_0$: $[7, 4, 3]$
  - $P_1$: $[1, 2, 2]$
  - $P_2$: $[6, 0, 0]$
  - $P_3$: $[0, 1, 1]$
  - $P_4$: $[4, 3, 1]$
- **Step-by-Step Work Vector Execution Trace**:
  1. $\text{Initial Work} = [3, 3, 2]$
  2. $P_1$ runs ($\text{Need}[1] = [1, 2, 2] \le [3, 3, 2]$) $\to \text{Work} = [3, 3, 2] + [2, 0, 0] = \mathbf{[5, 3, 2]}$
  3. $P_3$ runs ($\text{Need}[3] = [0, 1, 1] \le [5, 3, 2]$) $\to \text{Work} = [5, 3, 2] + [2, 1, 1] = \mathbf{[7, 4, 3]}$
  4. $P_4$ runs ($\text{Need}[4] = [4, 3, 1] \le [7, 4, 3]$) $\to \text{Work} = [7, 4, 3] + [0, 0, 2] = \mathbf{[7, 4, 5]}$
  5. $P_0$ runs ($\text{Need}[0] = [7, 4, 3] \le [7, 4, 5]$) $\to \text{Work} = [7, 4, 5] + [0, 1, 0] = \mathbf{[7, 5, 5]}$
  6. $P_2$ runs ($\text{Need}[2] = [6, 0, 0] \le [7, 5, 5]$) $\to \text{Work} = [7, 5, 5] + [3, 0, 2] = \mathbf{[10, 5, 7]}$
- **Resulting Safe Sequence**: $\mathbf{\langle P_1, P_3, P_4, P_0, P_2 \rangle}$ (System is Safe).

#### TC2–TC3: Request Vector, Resource Availability, and Reason for Denial / Unsafe Result
- **TC2 (Waiting/Deferred Request)**:
  - Process $P_0$ submits request: $\text{Request}[0] = [0, 4, 1]$.
  - $\text{Need}[0] = [7, 4, 3]$. Since $[0, 4, 1] \le [7, 4, 3]$, the request is legitimate.
  - $\text{Available} = [3, 3, 2]$. Comparison: Resource $B$ requested ($4$) exceeds Available ($3$).
  - **Reason**: The process cannot be granted resources immediately due to temporary pool depletion. The OS queues the process in a `WAITING` state without altering the Allocation matrix.
- **TC3 (Unsafe State Denial & State Rollback)**:
  - Process $P_4$ submits request: $\text{Request}[4] = [3, 3, 0]$.
  - $\text{Request} \le \text{Need}[4] ([4, 3, 1])$ and $\text{Request} \le \text{Available} ([3, 3, 2])$.
  - **Speculative Allocation**:
    - $\text{Available}' = [3, 3, 2] - [3, 3, 0] = [0, 0, 2]$.
    - $\text{Allocation}'[4] = [3, 3, 2]$, $\text{Need}'[4] = [1, 0, 1]$.
  - **Safety Test on Speculative State**: With $\text{Work} = [0, 0, 2]$, all processes have unmet positive requirements for resource $A$ or $B$:
    - $P_0$ needs $[7, 4, 3]$; $P_1$ needs $[1, 2, 2]$; $P_2$ needs $[6, 0, 0]$; $P_3$ needs $[0, 1, 1]$; $P_4$ needs $[1, 0, 1]$.
    - No process can run! $\text{Finish}$ vector remains False.
  - **Reason**: The resulting state is **UNSAFE**. The Banker's Algorithm denies the request and restores the original state. *(Note: An unsafe state is not deadlocked; it merely carries the risk of deadlock if all processes simultaneously claim maximums).*

#### TC4–TC5: Deadlock Relationships, Recovery / Prevention Action and Observed Effect
- **TC4 (Deadlock Detection & Recovery)**:
  - 3 Processes ($P_0, P_1, P_2$) and 2 Resources ($R_0, R_1$) with $\text{Available} = [0, 0]$.
  - $\text{Allocation}: P_0 = [1, 0], P_1 = [0, 1], P_2 = [1, 1]$.
  - $\text{Pending Request}: P_0 = [0, 1], P_1 = [1, 0], P_2 = [0, 1]$.
  - **Deadlock Relationship**: $P_0$ holds $R_0$ and waits for $R_1$; $P_1$ holds $R_1$ and waits for $R_0$; $P_2$ holds both and waits for $R_1$. Circular wait is established.
  - **Recovery Action**: Terminated victim process $P_2$. Its allocations $[1, 1]$ were reclaimed, updating $\text{Available} = [1, 1]$.
  - **Observed Effect**: $P_0$ satisfies its request, releases $[1, 0]$, allowing $P_1$ to finish. Deadlock cleared.
- **TC5 (Deadlock Prevention)**:
  - Applied **Hierarchical Resource Ordering Protocol**: Resources assigned monotonic ranks $F(R_0) = 1, F(R_1) = 2, F(R_2) = 3$.
  - Rule: A process holding $R_i$ may only request $R_j$ if $F(R_j) > F(R_i)$.
  - **Observed Effect**: When a process holding $R_1$ requested $R_0$, the system rejected the request immediately ($Rank\ 1 \le Rank\ 2$), mathematically preventing circular wait from ever forming.

#### TC6: Boundary and Invalid Inputs Checked
- **Negative Request**: Submitted $\text{Request} = [-1, 0, 0]$. Flagged: `"Input Error: Negative resource quantities are prohibited"`.
- **Request Exceeding Maximum Need**: $P_1$ requested $[5, 0, 0]$ when $\text{Need}[1] = [1, 2, 2]$. Flagged: `"Rejection: Process P1 requested more than its declared remaining Need"`.
- **Allocation Exceeding Maximum Claim**: Matrix validation caught $\text{Allocation} > \text{Maximum}$ upon input and disabled safety simulation until corrected.
- **Zero Available Resources**: Tested $\text{Available} = [0, 0, 0]$; all positive incoming requests correctly deferred to `WAITING` queue without errors.
- **Zero Remaining Need**: Process with $\text{Need} = [0, 0, 0]$ immediately executes and returns all held allocations to the Available vector.

---

## 4. AI Usage Record

- **AI Tools Used**: Google Gemini / Antigravity AI Assistant
- **Estimated Overall AI Usage**: **45%**
- **Basis of Estimate**: AI was utilized for boilerplate UI component generation in Streamlit, drafting algorithm pseudocode for Banker's rollback logic and Shoshani-Coffman detection, and generating initial test edge-case matrices. Students designed system architecture, verified all mathematical formulas on paper, debugged matrix state rollbacks, and implemented RAG graph rendering.

### Detailed Prompt and Verification Log

| AI Tool | Exact Prompt Used | Generated Content / File | Student Verification / Changes |
| :--- | :--- | :--- | :--- |
| **Google Gemini / Antigravity** | *"Generate Streamlit boilerplate with multi-tab layout and configurable pandas dataframe matrix editors for Banker's Algorithm."* | Initial layout structure in `app.py` with `st.data_editor` and tab definitions | Verified UI layout; adjusted data schemas and customized CSS styling for clean responsiveness. |
| **Google Gemini / Antigravity** | *"Implement Banker's Safety Algorithm and Resource-Request algorithm in Python with step-by-step Work vector logging and state rollback."* | `bankers_safety_algorithm()` and `request_resource()` functions in `app.py` | Manually solved TC1 and TC3 on paper; verified exact mathematical match with algorithm execution traces. Fixed rollback logic to restore original state. |
| **Google Gemini / Antigravity** | *"Provide Python logic for Shoshani-Coffman multi-instance deadlock detection and generate Graphviz DOT script for RAG visualization."* | `deadlock_detection_multi_instance()` and `generate_rag_dot()` functions | Verified RAG graph syntax; added dynamic node coloring (red for deadlocked processes, green for allocations, dashed for requests). |
| **Google Gemini / Antigravity** | *"Design comprehensive test scenarios (TC1 to TC6) covering safe state, deferred requests, unsafe requests, circular deadlock, prevention, and edge cases."* | Test matrix parameters and boundary condition checks in Tab 5 | Executed each scenario on the live dashboard, verified output logs, and compiled results into the Analytical Results Table. |

---

## 5. Submission Declaration

We confirm that the links, source code, test-case results, AI usage percentage, and prompts reported above accurately represent our group submission. We have independently verified the correctness of the dashboard outputs and understand the complete implementation.

- **Group Representative**: Ayush Jugseniye
- **Date**: 21 November 2026
- **Group Members**:
  1. Ayush Jugseniye (PRN: 202401040128 | Roll No: 32)
  2. Tejas Jaju (PRN: 202401040124 | Roll No: 29)
  3. Haider Shikalgar (PRN: 202401040272 | Roll No: 70)
