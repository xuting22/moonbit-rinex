# Frozen 0.1 scope

A pure MoonBit incremental line checker for RINEX 3.03/3.04/3.05/4.00/4.01 OBS, flags 0/1. Version selection is explicit; no NAV, RINEX2, compressed input or events 2–6. Input must be converted to ordinary ASCII RINEX by an external declared tool first.

Core contracts: fixed header/observation fields, per-system observation counts with continuation, finite numeric fields and LLI/SSI syntax, declared epoch/satellite record count, duplicate SV, exact 100ns time/order/gaps, header/actual endpoint and satellite total comparison. Short observation records omit trailing blanks as allowed by format; interior fixed columns remain fixed. No receiver scaling/correction, cycle-slip inference, positioning or satellite visibility expected-denominator claims.

Optional explicit EPN daily/hourly profile uses GPST with 30s lattice and whole supplied day/hour. It checks delivered epochs rather than individual satellite rising/setting. It reports unsupported / malformed separately from completely inspected with policy findings. It is a subset of EPN operational requirements, not EPN certification.

Memory: retain only header/counters/current-epoch SVs plus 128 issue examples, never full file/epoch archive. Limits: 8192 ASCII chars per line, 512MiB logical input, 4096 header lines, 96 observation types/system, 1m epochs and 2m satellite records. Fixed seed not relevant; deterministic report and exact decimal timestamps. Checker poisons after parse/capacity failure and seals on finish.

Delivery evidence: full official BRUX 2026-09-23 converted by RNXCMP4.2.0, CC-BY4.0 attributed small prefix for offline example, independent Python oracle, mutation cases (record truncation, epoch missing/duplicate/out of order, metadata mismatch, extra fields, unsupported event and capacity failure), JS/Wasm consumer. Existing GeoRust/gnss-js/EPN QC acknowledged; no novel-algorithm/ecosystem-empty claim.

Only GPS time is inspected; calendar range 1980..2100. Observation-code grammar is checked, not every constellation-specific valid signal combination. Nonempty/blank/zero are raw slot statistics, not valid measurement percentages. PRN detail values, scale factors and correction metadata are not interpreted.
