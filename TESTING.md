# Testing Guide

This guide describes a controlled validation workflow for the Ubuntu 24.04 LTS STIG Automation Tool.

## Test Environment

Use a disposable Ubuntu 24.04 LTS virtual machine or an approved staging host. Remediation changes packages, services, and system configuration. Do not run remediation on production without review and a recovery plan.

Required tools and access:

- Ubuntu 24.04 LTS
- Python 3.8 or newer
- `sudo` access for local remediation
- SSH access for remote testing
- A VM snapshot or equivalent rollback method

## Prepare the Environment

```bash
git clone <repository-url>
cd <repository-name>
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
chmod +x quick_start.sh run_with_sudo.sh
```

Alternatively, run `./quick_start.sh` to create `.venv`, install dependencies, create output directories, and set script permissions.

## Local Check-Only Test

Run an assessment without changing system configuration:

```bash
./run_with_sudo.sh --mode local --check-only
```

Verify that:

- The configured rules are loaded successfully.
- Each check returns `PASS`, `FAIL`, or `ERROR`.
- A compliance summary is printed.
- HTML and JSON reports are created in `reports/`.
- A log file is created in `logs/`.

Inspect the latest generated files:

```bash
ls -lt reports/ logs/
```

## Local Remediation Test

After reviewing the check-only report, run remediation on the test host:

```bash
./run_with_sudo.sh --mode local --auto-remediate
```

Confirm that the tool:

1. Performs a pre-remediation check.
2. Runs remediation only for failed rules.
3. Re-validates remediated rules.
4. Generates pre- and post-remediation reports.
5. Records command output and failures in the log.

For a focused test, provide a comma-separated rule list:

```bash
./run_with_sudo.sh --mode local \
  --rules UBTU-24-100010,UBTU-24-100800 \
  --check-only
```

## SSH Test

Verify connectivity independently before running the tool:

```bash
ssh -i ~/.ssh/id_rsa ubuntu@192.168.1.100 "echo SSH_OK"
```

Run a remote check-only assessment. Reports and logs are generated on the machine running the tool:

```bash
source .venv/bin/activate
python main.py \
  --mode ssh \
  --host 192.168.1.100 \
  --user ubuntu \
  --key ~/.ssh/id_rsa \
  --check-only
```

Only test remote remediation on an approved staging host:

```bash
python main.py \
  --mode ssh \
  --host 192.168.1.100 \
  --user ubuntu \
  --key ~/.ssh/id_rsa \
  --auto-remediate
```

Never place private keys or passwords in this repository, shell history, or documentation.

## Validation Checklist

### Functional

- [ ] `python main.py --help` exits successfully.
- [ ] Local check-only mode completes without remediation.
- [ ] Rule filtering checks only the requested IDs.
- [ ] Local remediation performs a post-check.
- [ ] SSH check-only mode connects and returns results.
- [ ] HTML and JSON reports are valid and readable.
- [ ] Logs contain the target host, execution mode, and rule summary.

### Safety

- [ ] Testing uses a disposable VM or staging host.
- [ ] A rollback snapshot exists before remediation.
- [ ] SSH configuration is validated before service restart.
- [ ] No credentials, private keys, logs, or generated reports are committed.

## Troubleshooting

### Missing dependencies

```bash
source .venv/bin/activate
python -m pip install -r requirements.txt
```

### Permission denied

```bash
chmod +x quick_start.sh run_with_sudo.sh
chmod +x scripts/checks/*.sh scripts/remediation/*.sh
```

### Debug a single remediation script

```bash
sudo bash -x scripts/remediation/UBTU-24-XXXXX.sh
```

### SSH connection failure

```bash
ssh -vvv -i ~/.ssh/id_rsa ubuntu@192.168.1.100
sudo systemctl status ssh
sudo ufw status
```

Review the generated log with the configured log level before changing a remediation script.

## Test Evidence

For each test run, record:

- Ubuntu version and test host type
- Execution mode and command used
- Number of rules checked
- PASS, FAIL, and ERROR counts
- Pre- and post-remediation compliance rates
- Rules requiring manual intervention
- Relevant log or report filenames
