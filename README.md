# k6-lab

ITDS362 Software Quality Assurance and Testing — Lab: Performance Testing with Grafana K6.

K6 scripts run against [QuickPizza](https://quickpizza.grafana.com), Grafana's public demo site for K6 practice.

- Part 0: K6 basics (`01-first-test.js` to `08-bug-hunt.js`, `explore.js`)
- Part A: a user journey (`journey.js`) run as smoke, average-load, stress, spike and soak tests (`a1-smoke.js` to `a5-soak.js`), with HTML reports in `reports/`
- Results and Part B answers: [RESULTS.md](RESULTS.md)

Run any script with `k6 run <script>.js`. Never more than 5 virtual users against QuickPizza.
