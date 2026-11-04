#!/usr/bin/env python3 """ 
Validates reachability across local subnet routes (192.168.0.0/24), the static Apple TV subnet router (192.168.2.1/32), and tailnet endpoints.
Usage: python3 tailscale_gateway_audit.py """
import subprocess import sys import os import platform
Define diagnostic targets
TARGETS = [ {"name": "Apple TV Subnet Router Node", "host": "192.168.2.1", "critical": True}, {"name": "Primary LAN Gateway", "host": "192.168.0.1", "critical": True}, {"name": "Tailnet FQDN", "host": "hale-werner.ts.net", "critical": False}, {"name": "Custom Domain Endpoint", "host": "eastfed.com", "critical": False}, {"name": "DNS-over-TLS Resolver", "host": "1.1.1.1", "critical": True}, ]
def ping_host(name: str, host: str) -> bool: """Execute a ping test based on OS platform.""" param = "-n" if platform.system().lower() == "windows" else "-c" timeout_param = "-w" if platform.system().lower() == "windows" else "-W"
cmd = ["ping", param, "2", timeout_param, "2", host]
try: result = subprocess.run( cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True ) if result.returncode == 0: print(f"[PASS] {name:<30} ({host}) is reachable.") return True else: print(f"[FAIL] {name:<30} ({host}) is unreachable.") return False except Exception as e: print(f"[ERROR] Executing ping for {name} ({host}): {e}") return False
def check_tailscale_status(): """Verify Tailscale local daemon status.""" print("\n--- Tailscale CLI Status Check ---") try: result = subprocess.run( ["tailscale", "status"], stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True ) if result.returncode == 0: print("[INFO] Tailscale daemon is active.") lines = result.stdout.strip().split('\n') for line in lines[:5]: print(f" {line}") else: print("[WARN] Tailscale daemon returned non-zero status or is not running.") except FileNotFoundError: print("[WARN] Tailscale CLI binary not found in system PATH.") except Exception as e: print(f"[ERROR] Checking Tailscale status: {e}")
def main(): print("=") print(" Tailscale Subnet Router & Gateway Health Audit ") print("=")
check_tailscale_status()
print("\n--- Network Target Ping Audit ---") passed = 0 critical_failed = False
for target in TARGETS: status = ping_host(target["name"], target["host"]) if status: passed += 1 elif target["critical"]: critical_failed = True
print("\n-------------------------------------------------------") print(f"Summary: {passed}/{len(TARGETS)} targets reachable.")
if critical_failed: print("[ALERT] One or more critical gateway targets failed reachability check.") sys.exit(1) else: print("[SUCCESS] All primary subnet and gateway checks completed.") sys.exit(0)
if name == "main": main()
