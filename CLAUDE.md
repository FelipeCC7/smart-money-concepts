# Smart Money Concepts - Claude Code Guide

## Project Purpose

Python library implementing Smart Money Concepts (SMC) trading indicators and analysis tools for algorithmic trading.

## Directory Structure

```
smart-money-concepts/
├── smartmoneyconcepts/      # Main Python package
│   ├── __init__.py
│   └── [indicator modules]
├── tests/                   # Test suite
│   └── [test files]
├── integration_steps/       # Integration documentation
├── .github/                 # GitHub workflows
├── setup.py                 # Package setup
├── README.md                # Documentation
├── LICENSE                  # MIT License
├── CODE_OF_CONDUCT.md       # Community guidelines
└── CONTRIBUTING.md          # Contribution guidelines
```

## What is Smart Money Concepts?

SMC is a trading methodology that identifies institutional trading patterns:
- **Order Blocks**: Areas of institutional buying/selling
- **Fair Value Gaps**: Price imbalances
- **Liquidity**: Stop-loss clusters
- **Market Structure**: Higher highs, lower lows
- **Break of Structure (BOS)**: Trend changes

## Installation

```bash
pip install -e .
```

## Usage

```python
from smartmoneyconcepts import smc

# Example: Identify order blocks
order_blocks = smc.order_blocks(df)

# Example: Find fair value gaps
fvg = smc.fair_value_gaps(df)

# Example: Market structure
structure = smc.market_structure(df)
```

## Integration with Algotrading

This library is used by:
- `algotrading_development/` - Strategy development
- `algotrading_production/` - Production trading

### Integration Path
```
smart-money-concepts (library)
        ↓
algotrading_development (R&D)
        ↓
algotrading_production (live)
```

## Running Tests

```bash
# Run all tests
pytest tests/

# Run with coverage
pytest --cov=smartmoneyconcepts tests/
```

## Key Indicators

| Indicator | Function | Description |
|-----------|----------|-------------|
| Order Blocks | `order_blocks()` | Institutional S/R zones |
| FVG | `fair_value_gaps()` | Price imbalances |
| Liquidity | `liquidity()` | Stop-loss clusters |
| BOS | `break_of_structure()` | Structure breaks |
| CHoCH | `change_of_character()` | Trend reversals |

## Contributing

See `CONTRIBUTING.md` for guidelines:
1. Fork the repository
2. Create feature branch
3. Write tests
4. Submit PR

## Dependencies

- pandas
- numpy
- (check setup.py for full list)

## Notes

- Open source library (MIT License)
- Part of larger algotrading ecosystem
- Focus on clean, testable code
- Document all indicator logic
