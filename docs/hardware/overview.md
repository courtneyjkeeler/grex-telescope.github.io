# Hardware Overview

The GReX hardware system has several "top level" components, which constitute
the entire system. These include the [feed antenna](feed.md) and low noise
amplifiers (LNA), the [frontend module](fem.md), the [digital backend](fpga.md),
and of course the server. The following diagrams lay out general overview of the
interconnections. Showing them all at once would be a bit much, so they're
broken down here into discrete kinds of signals.

## RF Signal Path

```mermaid
flowchart TD
    A[Feed]
    B[FEM]
    C[SNAP]
    D[Server]
    A --> L1[LNA]
    A --> L2[LNA]
    L1 -->|H Pol| B
    L2 -->|V Pol| B
    subgraph The Box
        B -->|H Pol| C
        B -->|V Pol| C
    end
    C -->|10 GbE| D[Server]
```

## Power Distribution

```mermaid
flowchart BT
    L1[LNA]
    L2[LNA]
    B[FEM]
    C[SNAP]
    S[Switching Supply]
    R[Linear Regulator]
    P[Raspberry Pi]
    M[Mains Power]
    V[Synthesizer]
    G[GPS Receiver]
    M -->|120-240V AC| S
    subgraph The Box
        R -->|6.5V DC| V
        P -->|"5V DC (USB)"| G
        S -->|12V DC| C
        S -->|12V DC| R
        R -->|6.5V DC| B
        C -->|5V DC| P
    end
    B --->|5.5V DC| L1
    B --->|5.5V DC| L2
```

## Clocks, References and Timing

```mermaid
flowchart BT
    B[FEM]
    V[Synthesizer]
    G[GPS Receiver]
    S[SNAP]
    A[GPS Antenna]
    subgraph The Box
        G -->|10 MHz| V
        G -->|PPS| S
        V -->|500 MHz| S
        V -->|1030 MHz| B
    end
    A --> G
```

## Monitor and Control

```mermaid
flowchart TB
    B[FEM]
    S[SNAP]
    P[Raspberry Pi]
    D[Server]
    subgraph The Box
        P <-->|UART| B
        S <-->|GPIO| P
    end
    P <--->|1 GbE| D
```
