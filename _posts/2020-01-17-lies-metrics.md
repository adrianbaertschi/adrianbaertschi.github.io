---
title: Lies, Damned Lies, and Metrics
---
Notes for the talk [Lies, Damned Lies, and Metrics](https://www.youtube.com/watch?v=goihWvyqRow) on GOTO 2019 conference.

# Purpose / Why do we need metrics?
- Planning, Progress (avoid)
  - Get context
  - Know when we are done
  - Predict issues
  - Hindsight on issues
  - Examples: Burndown Chart, Velocity, Cumulative flow, WIP
- CI/CD (avoid)
  - Fast feedback
  - Examples: Build / deploy / test speed, PR Approval time
- Politics (avoid)
  - Show / convince / avoid management
  - Examples: $ wasted in waiting,  cost of fixing prod bug, number of issues
- Transformation (do it)
  - Influence behaviour
  - Measure impact of experiments
  - Pairing time, PR approve time
- Make a decision (do it)
  - Examples: Lead time, escaped bugs, delivered value


# Leading vs. Lagging indicators
- Leading
  - Can be influenced directly
  - "Input"
  - Try to influence future performance
  - Are not goals
- Lagging
  - Many factors at play
  - "Output"
  - Analyse past performance
  - Finally often: money made
  - => Use this one
- Example: Weight loss
  - Leading: Calorie intake, hours of sport
  - Lagging: kg lost

# Keep in mind
- Use more than one lagging indicator (to handle tradeoffs)
- Pair leading to lagging
- Only measure if you have a dilemma, know what to change