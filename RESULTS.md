# K6 lab results

## Part 0
- Step 0.1, http_req_duration: 328.63 ms (1 request, so avg = min = med = max = p(90) = p(95) = 328.63 ms). checks_succeeded 100.00% (1 out of 1), http_reqs 1, iterations 1.
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 75 requests, 3.67 requests/s, p(95) = 319.39 ms (0 failed, checks 75/75). The rate stays below 5 because one VU does not send one request per second. One iteration is the request (avg 299.66 ms) plus `sleep(1)`, so iteration_duration averages 1.34 s. One VU therefore makes about 1 / 1.34 ≈ 0.75 requests per second, and 5 VUs make about 5 × 0.75 ≈ 3.7 requests per second, which matches the measured 3.67/s. The rate would reach 5/s only if the response took no time at all.
- Step 0.6, exit code of the normal run and of the run with P95=300: normal run exit code **0** (all three thresholds passed: checks 100.00%, p(95)=463.1ms < 1500, http_req_failed 0.00%). Run with `-e P95=300` exit code **99** (`✗ 'p(95)<300' p(95)=461.54ms`, K6 printed "thresholds on metrics 'http_req_duration' have been crossed"; the other two thresholds still passed).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: `07-requirement.js` uses `vus: 3`, `duration: '30s'`, `http_req_duration: ['p(95)<1000', 'p(99)<1500']` and `http_req_failed: ['rate<0.01']` (plus `summaryTrendStats` so the summary prints p(99)). All three rules passed: p(95) = 495.43 ms < 1000, p(99) = 584.87 ms < 1500, http_req_failed = 0.00% < 1% (65 requests), exit code 0. Then I changed one number, "95% within 1 second" to "95% within 0.4 seconds" (`p(95)<400`), and ran it again: `✗ 'p(95)<400' p(95)=482.15ms`, the other two rules still passed, and the exit code was **99**. The script was set back to `p(95)<1000` afterwards.
- Step 0.8, what was wrong, and how the checks showed it:

## Part A results
| Step | Test type    | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1   | Smoke        | 1     |       |       |     |        |        |
| A2   | Average load | 3     |       |       |     |        |        |
| A3   | Stress       | 5     |       |       |     |        |        |
| A4   | Spike        | 5     |       |       |     |        |        |
| A5   | Soak         | 2     |       |       |     |        |        |

## Part B
1.
2.
3.
4.
5.
6.
7.
8.
9.
