# Notes index

These files mirror the six domains of the **CCNA 200-301 v1.1** exam blueprint, with the weighting of each shown. The Notion workspace remains the full day-by-day set following Jeremy's IT Lab; these are the condensed, review-oriented version kept in the repo so it is searchable, diffable, and readable offline.

| # | File | Domain | Weight |
|---|---|---|---|
| 1 | [01-network-fundamentals.md](01-network-fundamentals.md) | Network Fundamentals | 20% |
| 2 | [02-network-access.md](02-network-access.md) | Network Access | 20% |
| 3 | [03-ip-connectivity.md](03-ip-connectivity.md) | IP Connectivity | 25% |
| 4 | [04-ip-services.md](04-ip-services.md) | IP Services | 10% |
| 5 | [05-security-fundamentals.md](05-security-fundamentals.md) | Security Fundamentals | 15% |
| 6 | [06-automation-and-programmability.md](06-automation-and-programmability.md) | Automation and Programmability | 10% |

Plus two references that cut across domains:

- [07-commands-cheat-sheet.md](07-commands-cheat-sheet.md) — every IOS command grouped by task, in the style of the "Commands" section the note-generation prompt asks for.
- [08-subnetting-reference.md](08-subnetting-reference.md) — mask table, the four-step method, VLSM and summarization worked examples.

## Suggested review order

Fundamentals and subnetting first, because everything else assumes them. Then Network Access (VLANs, STP, EtherChannel), then IP Connectivity (routing table, static, OSPF, FHRP), which together are 45% of the exam. IP Services and Security are largely configuration recall. Automation is the smallest domain and the most self-contained, so it is a good one to leave until last.

Work the [labs](../labs/) alongside domains 2 and 3 rather than after them. The "things to break on purpose" section in each lab is where the troubleshooting marks come from.
