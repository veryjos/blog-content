## duck-hunt-vr

A **Virtual Reality** remake of Duck Hunt made for **Global Game Jam 2016**.

This is mostly interesting because it predates any **commercial hand-tracking** solutions for virtual reality headsets. Instead, I stuck a **Razer Hydra** on top of my HMD (Oculus DK1 at the time) with a small 3D-printed adapter thing.

The biggest challenge of this setup was compensating for the differences in latency between the two systems, since turning your head physically changed the orientation of the base station.

Features:

  - Head/head tracking with **6 degrees of freedom**
  - Skeet shooting minigame
  - Progressive difficulty increments
  - VR-friendly HUD
  - Cool retro effects

Written in **C++** in **Unreal Engine 4**. I later added **OpenVR** support so people with commercial headsets like the **HTC: Vive** could play.
