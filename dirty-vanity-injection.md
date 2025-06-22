# 🧠 Dirty Vanity

**Dirty Vanity** is a novel code injection technique that allows attackers to stealthily execute malicious payloads in Windows by evading Endpoint Detection & Response (EDR) systems.



## 🚨 What is Dirty Vanity?

Dirty Vanity splits traditional code injection steps between two processes, using Windows process forking to confuse EDRs and execute malicious code in a cloned process—without alerting security tools.


## 🤔 What is EDR?

**EDR (Endpoint Detection & Response)** is a system that monitors a machine for suspicious activity, especially behaviors like memory allocation, code writing, and code execution within trusted processes.


## 🧪 Traditional Injection vs. Dirty Vanity

### 🧬 Traditional Code Injection (EDR can detect):
1. Allocate memory in target process
2. Write malicious code into it
3. Execute the code

### 🪞 Dirty Vanity (EDR gets confused):
1. Inject code into a normal process (e.g., `explorer.exe`)
2. **Fork** the process using a Windows API (`RtlCreateProcessReflection`)
3. Run the malicious code in the **forked process** (not the original)

✅ Original process remains untouched  
❌ EDR doesn’t see the code execution path


## 🛡️ Why is it Dangerous?

Most EDRs:
- Only monitor the original process
- Don’t trace memory changes into cloned processes
- Fail to detect when malicious code is run inside a **forked clone**

As a result:
- ✅ Malicious code executes successfully
- ✅ Original process looks clean
- ❌ EDR doesn’t raise an alert



## 🏗️ Real-World Example

### Goal: Dump credentials from `lsass.exe`
- **Traditional Way:** Touching `lsass.exe` alerts EDR.
- **Dirty Vanity:** Clone `lsass.exe`, dump credentials from the **clone**—EDR doesn’t detect the operation.


## ⚙️ Requirements to Run Dirty Vanity

- Permission to write to another process
- Permission to fork a process (`RtlCreateProcessReflection`)
- Basic C/C++ and Windows internals knowledge


## 🔍 Detection & Future Mitigation

Currently, most EDRs are blind to this. Future improvements might include:
- Tracking forks (e.g., `RtlCreateProcessReflection`)
- Linking parent-write events with clone-execution events
- Monitoring memory in forked processes


## 🧪 Dirty Vanity Lab Setup Guide

> ⚠️ **Disclaimer:** Never test this on a real or production system. Use an offline virtual machine (VM) only.

### ✅ What You’ll Need

| Tool                              |          Purpose                      |
|-----------------------------------|---------------------------------------|
| VirtualBox / VMware               | Virtual environment                   |
| Windows 10/11 VM                  | Victim/test system                    |
| Kali Linux VM *(optional)*        | To create payloads (e.g., `msfvenom`) |
| Visual Studio                     | Compile injector code                 |
| x64dbg / OllyDbg                  | Debugging shellcode                   |
| Process Hacker / Process Explorer | Observe forks and memory              |
| Windows SDK / Sysinternals Suite  | Inspect system behavior               |


## 📦 Download the Dirty Vanity Code

```bash
git clone https://github.com/deepinstinct/Dirty-Vanity.git
```

---

## 🧱 Lab Setup Steps

### 1. Set Up Victim Process

Use a known process like:
- `notepad.exe`
- `explorer.exe`
- or a custom dummy process

### 2. Compile Injector

- Open `DirtyVanity.cpp` in Visual Studio
- Link against `ntdll.lib`
- Use x64 Debug/Release mode

### 3. Choose Your Shellcode

Start simple:

```cpp
MessageBoxA(NULL, "Dirty Vanity Lab", "Test", MB_OK);
```

Or use a template:

🔗 [windows_x64_shellcode_template](https://github.com/rainerzufalldererste/windows_x64_shellcode_template)


## 🔎 Observation Tools

| Tool           |         Watch For                     |
|----------------|---------------------------------------|
| x64dbg         | Shellcode execution in forked process |
| ProcMon        | Memory & process activity             |
| Process Hacker | Process tree showing forks            |
| Event Viewer   | Suspicious process activity           |


## 🧪 Test Scenario (Step-by-Step)

1. Launch `notepad.exe`
2. Run the compiled Dirty Vanity injector
3. In **Process Hacker**, observe:
   - A forked version of `notepad`
   - Code execution inside the fork
4. Optional: Set breakpoints in x64dbg to trace execution


## 🔒 Safety Tips

- Take VM **snapshots** before testing
- Disable **internet access** on VMs
- Monitor all behavior with **ProcMon**
- **Never** test on a host/work machine


## 📌 In One Sentence

> Dirty Vanity is a stealthy process injection technique that bypasses EDR by running malicious code in a **forked clone** of a process, instead of the original target.


## 🔗 References

- Official GitHub Repo: [deepinstinct/Dirty-Vanity](https://github.com/deepinstinct/Dirty-Vanity)
- Shellcode Template: [windows_x64_shellcode_template](https://github.com/rainerzufalldererste/windows_x64_shellcode_template)
