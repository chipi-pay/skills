# SKU Categories

Chipi SKU marketplace supports 14 categories of digital products and services.

**Geographic availability: Mexico only.** More countries coming soon.

## Available Categories

| # | Category | Examples |
|---|----------|----------|
| 1 | telephony | Phone top-ups, data plans, prepaid minutes |
| 2 | electricity | Bill payments, prepaid electricity |
| 3 | internet | ISP payments, data packages |
| 4 | streaming | Netflix, Spotify, Disney+, YouTube Premium |
| 5 | gift cards | Amazon, iTunes, Google Play, Steam |
| 6 | gaming | PlayStation, Xbox, Nintendo, Fortnite V-Bucks |
| 7 | transport | Uber, DiDi, Cabify credits |
| 8 | toll payments | Highway toll payments |
| 9 | TV | Cable TV, satellite TV payments |
| 10 | mobility | Scooter, bike-share credits |
| 11 | government | Tax payments, permit fees |
| 12 | insurance | Premium payments |
| 13 | education | Tuition, course fees |
| 14 | health | Health insurance, clinic payments |

## Reference Field

Each SKU type requires a specific reference format:
- **telephony:** Phone number (10 digits for Mexico)
- **electricity:** Meter/account number
- **internet:** Service account ID
- **streaming:** Account email or ID
- **gift cards:** Recipient email (optional)

The SKU object includes a `referencePattern` regex for validation.
