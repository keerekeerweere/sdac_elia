# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant custom component (`sdac_elia`) that fetches Belgian SDAC (Single Day Ahead Coupling) electricity prices from Elia (Belgium's TSO). Distributed via HACS.

## Commands

### Testing
```bash
pytest                    # Run all tests with coverage
pytest tests/test_init.py # Run a single test file
```

### Code Quality
```bash
pre-commit run --all-files   # Run all pre-commit hooks (black, ruff, mypy, codespell)
```

## Architecture

### Component Flow

```
Config Flow → __init__.py → sensor.py → Coordinator → Elia API
```

**Key files:**
- `custom_components/sdac_elia/coordinator.py` — Core logic. Polls every 1 minute. Fetches QH prices from `https://griddata.elia.be/eliabecontrols.prod/interface/Interconnections/daily/auctionresultsqh/{date}`. Fetches today at startup, tomorrow after 13:00 local time.
- `custom_components/sdac_elia/sensor.py` — Three sensors: current SDAC price, custom price formula, custom injection tariff.
- `custom_components/sdac_elia/config_flow.py` — HA config flow UI for initial setup and options reconfiguration.
- `custom_components/sdac_elia/const.py` — Domain name and config/data key constants.

### Price Calculations

The API returns prices in **€/MWh**. User-facing formulas use **€/kWh** (÷1000 conversion factor).

- Current SDAC: matches UTC datetime to nearest 15-minute boundary in the fetched QH data
- Custom price: `(factor × SDAC + fixed) × 1000` → converts to €/kWh
- Custom injection: `(factor × SDAC − fixed) × 1000` → converts to €/kWh

### Testing Framework

Uses `pytest-homeassistant-custom-component`. The `tests/conftest.py` enables custom integrations. CI runs hassfest (HA manifest validation) and HACS validation via GitHub Actions.
