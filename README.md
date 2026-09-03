# F1 2026 Belgian GP: Energy Deployment Analysis

A personal project that explores the way in which F1 teams and drivers handled their battery deployment strategy (MGU-K) in qualifying of the 2026  Belgium Grand Prix. Public telemetry data along with many approximate models have been used instead of physical constants. 

The public telemetry doesn't provide the SOC of the battery or the power output of the MGU-K system, therefore everything is based on the speed, throttle, brake, and the gear trace information.

## What's here

- `f1_belgium_2026_econometric_analysis.ipynb` - the whole pipeline, data pull through results

Running it creates a `Spa 2026 figures/` folder with the plots (resistance fits, regime probabilities, driver effects, teammate delta charts).

## What it does

- Regresses the coasting periods of each individual car separately in order to compute drag (CdA) and rolling resistance (Crr), rather than using a generic number for all cars
- Distinguishes between harvest/neutral/deploy using a Markov-switching model of acceleration, this attributing a probability to each data point, rather than setting a fixed level of throttle as the boundary
- Computes a theoretical optimal lap using a stochastic frontier analysis per mini-sector, combining the optimal mini-sector across all competitors rather than simply taking the fastest single lap time
- Attempts to separate driver ability from car capability by including two-way fixed efforts (mini-sectors + team + driver)
- Redistributes the deployed energy of an individual driver around the lap in order to identify the points where the investment would have provided the largest gains in terms of seconds per MJ

## Findings

Antonelli got pole by over three-tenths, with the fastest six cars covered by just 0.534s. The engines used by McLaren and Mercedes belong to the same family; this proved significant as both Piastri and Russell suffered from deployment issues caused by the same self-learning PU software. This can be seen in the probability tables as well as in the news. Hadjar's lap was mostly a tow job for Verstappen after an already-fixed grid penalty. Hulkenberg lost Q2 to a hydraulic failure, both visible in the data once looked at. 

## What's broken (and what's still broken)

The initial attempt generated output that violated basic physical laws for all simulations: no resistances were calculated properly. All drivers exceeded the 8.5 MJ/lap deployment limits. Theoretical fastest lap time turned out to be almost 11 seconds faster than any real driver's performance. Two-way FE calculations gave the standard error of 1e17. Five distinct bugs were identified:

1. Distance interpolation problem caused by small non-monotonic fluctuations in the telemetry
2. Brake channel was evaluated based on a fixed scale, which didn't match the data
3. Lack of the elevation variable resulted in using the 100-meter elevation of Spa as an extra part of the rolling resistance
4. Total power of both ICE and MGU-K was compared with the deployment limit for MGU-K only
5. Unrecognised collinearity in the two-way FE formula, since team number is fully determined by the driver number in this particular session

Of these, four are fixed and compared o those from the synthetic self-test in the notebook. There is one left open: Norris's resistance fit is still wrong, but a bootstrap and a track-location check both rule out the obvious explanations (small sample, weird coast-point location), so it's a real, unexplained pattern rather than noise. Live guess is the active-aero system closing right as he lifts, contaminating the coast-phase points. Haven't confirmed it yet.
