---
name: Map and navigate a robot with the OM1 ROS2 SDK
description: Build a SLAM map, start Nav2, and send a robot to a pose using the OM1 ROS2 SDK Premium REST APIs.
api: OM1 ROS2 SDK Premium REST API
base_url: http://<robot-host>:5000 (Orchestrator), http://<robot-host>:5001 (Nav2)
operations:
  - POST /start/slam
  - POST /maps/save
  - POST /start/nav2
  - POST /api/move_to_pose
  - GET /api/nav2_status
auth: on-robot REST APIs (Enterprise / Premium Features)
---

# Map and navigate a robot with the OM1 ROS2 SDK

The OM1 ROS2 SDK exposes two on-robot REST APIs (Premium/Enterprise):
the **Orchestrator API** on port 5000 (SLAM, Nav2, patrol, charging, maps) and
the **Nav2 API** on port 5001 (pose, goals, localization).

## Steps

1. **Start base control**, then map: `POST /start/slam` on the Orchestrator API
   (must not have Nav2 running). Drive the robot to build the occupancy grid.

2. **Save named locations** while mapping with `POST /maps/locations/add/slam`
   (`map_name`, `label`), then **save the map** with `POST /maps/save`
   (`map_name`) before stopping SLAM.

3. **Switch to navigation.** Stop SLAM, then `POST /start/nav2` with the saved
   `map_name`. The robot localizes within the map.

4. **Send a goal.** `POST /api/move_to_pose` (Nav2 API, port 5001) with target
   `position` and `orientation` (quaternion). It returns immediately.

5. **Monitor progress** with `GET /api/nav2_status` until the goal `status` is
   `SUCCEEDED` (watch for `ABORTED`/`CANCELED`); use `GET /api/amcl_variance` to
   confirm localization confidence.

## Rules

- SLAM and Nav2 are mutually exclusive — stop one before starting the other
  (a 400 is returned otherwise).
- Charging/docking and patrol endpoints are Unitree Go2-only and require Nav2
  to be running.
- Responses use `{"status": "success"|..., "message": ...}`; treat non-`success`
  status and 400/404/500 as failures and re-check `GET /status`.
