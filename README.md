# Ubuntu 24.04 LTS STIG Automation

A Python and Bash security automation tool for auditing and remediating Ubuntu 24.04 LTS systems against DISA STIG controls.

This project demonstrates practical DevOps and security automation skills: repeatable checks, controlled remediation, local and SSH execution, structured reporting, and operational logging.

## Project Highlights

- **32 configured controls** with separate check and remediation scripts
- **CAT I, CAT II, and CAT III** security checks
- **Check-only mode** for safe assessment before making changes
- **Auto-remediation with re-validation** and pre/post compliance reports
- **Local and SSH execution** for single-host and remote assessments
- **HTML and JSON reports** for review and audit evidence
- **Rotating logs** and configurable execution paths
- **Backup-aware remediation** to reduce operational risk

## Architecture

```text
CLI (main.py)
    |
    +--> Configuration (config/settings.yaml, config/stig_rules.json)
    |
    +--> Executor (local shell or SSH)
    |
    +--> Checker (STIG validation scripts)
    |
    +--> Remediator (approved remediation scripts)
    |
    +--> Reporter (HTML and JSON output)
```

The execution pipeline is:

```text
Load rules -> Pre-check -> Pre-report -> Remediate -> Post-check -> Post-report
```

## Repository Structure

```text
.
├── main.py                    # CLI entry point and orchestration pipeline
├── requirements.txt           # Python dependencies
├── config/
│   ├── settings.yaml          # Runtime paths and logging configuration
│   └── stig_rules.json         # Rule metadata and severity mapping
├── lib/
│   ├── checker.py              # Check execution and result collection
│   ├── executor.py             # Local and SSH command execution
│   ├── logger.py               # Console and rotating-file logging
│   ├── remediator.py           # Remediation orchestration
│   └── reporter.py              # HTML and JSON report generation
├── scripts/
│   ├── checks/                 # STIG validation scripts
│   └── remediation/            # Remediation scripts and shared helpers
├── reports/                    # Generated reports (ignored by Git)
├── logs/                       # Runtime logs (ignored by Git)
├── quick_start.sh              # Dependency and permission setup
├── run_with_sudo.sh            # Privilege-aware execution wrapper
├── TESTING.md                  # Local and SSH testing guide
└── .gitignore
```

## Quick Start

> Run this tool on a disposable lab VM or an approved test host first. Remediation changes system configuration and requires root privileges.

### Requirements

- Ubuntu 24.04 LTS
- Python 3.8 or newer
- `sudo` access for local remediation
- SSH access and a key or password for remote mode

### Installation

```bash
git clone <repository-url>
cd <repository-name>

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
chmod +x quick_start.sh run_with_sudo.sh
```

### Run a safe assessment

```bash
./run_with_sudo.sh --mode local --check-only
```

### Run assessment and remediation

```bash
./run_with_sudo.sh --mode local --auto-remediate
```

### Assess a remote Ubuntu host

```bash
python3 main.py \
  --mode ssh \
  --host 192.168.1.100 \
  --user ubuntu \
  --key ~/.ssh/id_rsa \
  --check-only
```

See all available options:

```bash
python3 main.py --help
```

## Output

Each run can produce:

- `reports/stig_report_<timestamp>.html` for human review
- `reports/stig_report_<timestamp>.json` for machine-readable results
- `logs/stig_<timestamp>.log` for troubleshooting and audit history

Generated reports and logs are excluded from version control.

## Engineering Practices Demonstrated

- Configuration-driven execution instead of hard-coded paths
- Separation of orchestration, execution, checking, remediation, and reporting
- Idempotent shell-based controls where supported
- SSH automation with Paramiko
- Structured output suitable for audit evidence or later CI integration
- Explicit pre-check and post-check validation
- Least-surprise operational flow with check-only mode by default

## Scope and Limitations

This project currently targets Ubuntu 24.04 LTS and the controls defined in `config/stig_rules.json`. Test remediation in a controlled environment before production use. The tool is an automation project and does not replace security review, change management, or a formal compliance assessment.

## Documentation

- [Detailed testing strategy](TESTING.md)
- [STIG rule definitions](config/stig_rules.json)
- [Runtime configuration](config/settings.yaml)

## License

MIT License. See the repository license file when published.
