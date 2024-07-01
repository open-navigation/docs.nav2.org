# Visualization Guide

While using, debugging, or developing the project, you'll likely need to visualize internal and external states of a robotic system, such as transform trees, sensor data, simulated ground truths, etc. Although its easiest to consider dev containers as headless environments, there are still yet a myriad of methods to display visualizations using local, remote, or web-based graphical user interfaces.

![Nav2 running in Dev Container with Foxglove and GzWeb running as PWAs](/images/devcontainers/nav2_foxglove_gzweb_pwa.png)

For example, the screenshot above depicts a simulated demo, launched from the `nav2_simple_commander` package, running from inside a dev container, opened via VS Code, with Foxglove Studio and GzWeb respectively visualizing Nav2 and Gazebo from split web browser windows.

:::{hint}
This demo can be launched from a headless dev container by sourcing the overlay workspace, and disabling the gzclient and rviz GUIs:
``` bash
source $OVERLAY_WS/install/setup.bash
source /usr/share/gazebo/setup.sh
GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:$(find /opt/ros/$ROS_DISTRO/share \
  -mindepth 1 -maxdepth 2 -type d -name "models" | paste -s -d: -)
ros2 launch nav2_simple_commander security_demo_launch.py \
  use_rviz:=False headless:=True
```
:::

:::{seealso}
Checkout the Nav2 Simple Commander [Examples and Demos](/commander_api/index.rst) for more documentation on the depicted simulation.
:::

There are a few ways to visualize the project within dev containers, each with their own pros and cons. Ordered by increasing complexity:

- [**Web Apps**](#web-apps)
  - Simple and remotely accessible from any web browser, while providing a responsive interface
- [**Native Apps**](#native-apps)
  - Performative over local networks, while providing a native interface and GPU acceleration
- [**Desktop Stream**](#desktop-stream)
  - Flexible and remotely accessible from any client, while providing robust integration
- [**Display Socket**](#display-socket)
  - Performative and flexible for local development, while providing robust integration

The decision matrix below provides a rough comparison of the various options listed. Check each section for further details.

| Approach vs Utility | Web Apps | Native Apps | Desktop Stream | Display Socket |
|---------------------|----------|-------------|-------------|------------|
| Setup Simplicity    | 🟢        | 🟠           | 🟠           | 🔴          |
| Tooling Flexibility | 🟠        | 🟠           | 🟢           | 🟢          |
| Robust Integration  | 🟠        | 🔴           | 🟢           | 🟢          |
| Responsive UI/UX    | 🟢        | 🟢           | 🔴           | 🟠          |
| GPU Acceleration    | 🟢        | 🟢           | 🔴           | 🟠          |
| Remote Connectivity | 🟢        | 🟠           | 🟢           | 🔴          |
| Bandwidth Efficient | 🟠        | 🟢           | 🟠           | 🔴          |

> Legend: 🟢 Good | 🟠 Modest | 🔴 Poor

```{rst-class} content-collapse
```

(web-apps)=
## Web Apps

By far one of the easiest ways to quickly visualize the project is by using web client interfaces, e.g. Foxglove Studio and GzWeb. Such apps are accessible from any web browser, regardless of the host operating system or whether the dev container is running locally or remotely.

:::{seealso}
More details on Foxglove Studio and GzWeb can be found here:

- [Foxglove Studio](https://foxglove.dev/studio)
  - Robotics visualization and debugging with customizable layouts
- [Foxglove Bridge](https://github.com/foxglove/ros-foxglove-bridge)
  - Foxglove WebSocket bridge for ROS 1 and ROS 2
- [GzWeb](https://gazebosim.org/gzweb)
  - Web client for Gazebo simulation
:::

This is useful for:

- Remote Development
  - Enables deployment of headless or cloud based development environments
  - Applicable for onboarding contributors or coordinating across distributed teams
  - Necessary when local access to computing power or hardware resources are limited
- Education
  - Enables robotics curriculums, workshops, tutorials, or MOOCs using ROS
  - Applicable for sidestepping setup issues across varieties of student hardware
  - Necessary when class sizes are large or when instructor resources are limited

The respective utility for this approach can be summarized as follows:

- Setup Simplicity
  - Little to no client-side setup or prerequisites required, other than a web browser
  - Suitable for resource constrained devices, e.g. mobile phones or Android or iOS tablets
- Tooling Flexibility
  - Sufficient for common use cases, e.g. introspection, debugging, and education
  - Limited to newer web-based ecosystem, i.e. incompatible with RQt or RViz plugins
- Robust Integration
  - Support for most ROS message types and streaming of remote assets
  - Limited or experimental support for ROS actions and parameters
- Responsive UI/UX
  - Native like user interface with fluid control and visual feedback
  - Suitable for insulating user experience from network latency and bandwidth
- GPU Acceleration
  - Client-side rendering for 3D visualizations done in browser via WebGL
  - Suitable frame rates without fiddling with drivers or display settings
- Remote Connectivity
  - Usable via simple port forwarding for straight forward remote access
  - Suitable for tunneling without software defined networks or virtual interfaces
- Bandwidth Efficient
  - Lower minimal bandwidth requirements than streaming server rendered frame buffers
  - Limited to modest message rates/sizes based on network/bridge bandwidth/performance

### Example

After launching the Nav2 Simple Commander demo, as hinted above, you can then start the web app backends. This can be done using the included VS Code tasks via the Command Palette and selecting `Tasks: Run Task` command, then the `Start Visualizations` task.

:::{tip}
You can inspect how this [VS Code Task](https://code.visualstudio.com/docs/editor/tasks), and others included, work by viewing the `.vscode/tasks.json` file in the project repository.
:::

This will start the respective bridges and web servers used by Foxglove Studio and GzWeb inside the dev container, where supporting tools, such as VS Code can then be used to automatically forward the ports opened by these backends to the local client. You can then click on the `Open in Browser` icon for the  navigate to `Ports` view in the bottom panel, or by running the command `Ports: Focus on Ports View`.

:::{seealso}
More details on VS Code Remote Development can be found here:

- [Forwarding a port / creating SSH tunnel](https://code.visualstudio.com/docs/remote/ssh#_forwarding-a-port-creating-ssh-tunnel)
  - via the Remote Development extension from a local VS Code client
- [Forwarding ports in your codespace](https://docs.github.com/en/codespaces/developing-in-codespaces/forwarding-ports-in-your-codespace?tool=webui)
  - via supported Dev Container tooling from a remote Codespace
:::

![VS Code Ports View](/images/devcontainers/vscode_ports_view.png)

Then click on the `Open in Browser` icon to launch the respective web app from a new browser tab, or click on the `Preview in Editor` icon just to the right of it to launch the web app directly in a new VS Code editor tab instead. Respective ports for each web app are:

- Foxglove Studio: `8080`
- GzWeb: `9090`

You can then resize the browser window to your liking, and even move the tab to a separate window or monitor for a split screen view.

:::{tip}
You can also install Foxglove Studio and GzWeb as PWAs, allowing you to resize them like native apps separate from the browser. Just remember that PWAs are fixed to the URL they were installed from, so you'll need to reinstall if the URLs or port numbers ever change.
:::

:::{seealso}
More details on Progressive Web Apps installation can be found here:

- [Use Progressive Web Apps](https://support.google.com/chrome/answer/9658361)
  - with Google Chrome (i.e. on Computer or Android)
- [PWA Installation](https://web.dev/learn/pwa/installation)
  - with any browser (i.e. Firefox, or Safari on iOS)
:::

<!-- # TODO: expand
bridge
statefull
gracefull reconect -->

```{rst-class} content-collapse
```

(native-apps)=
## Native Apps

  - Pro: Leverage natively installed desktop applications for rendering, e.g. Rviz, Gzclient, etc.
  - Con: Fragile when missing or changing interface and resources, e.g. message types or 3D assets

host networking for ROS ports
 gzserver
native foxglove

resources and interfaces

```{rst-class} content-collapse
```

(desktop-stream)=
## Desktop Stream

  - Pro: Applicable when developing remotely over moderate yet finite network bandwidth
  - Con: Not performative or responsive, e.g.

bandwidth (large by more finite) vs resource usage

https://github.com/devcontainers/features/tree/main/src/desktop-lite

:::{seealso}
This approach is similarly used by other robotic web based services. More examples can be found here:

- [The Construct](https://www.theconstructsim.com)
  - An educational and training platform for robotics development and simulation
- [AWS RoboMaker](https://aws.amazon.com/robomaker)
  - A Continuos Integration and Deployment service for robotic applications and simulations
:::

:::{seealso}
- [Sunshine](https://github.com/LizardByte/Sunshine)
  - Self-hosted game stream host for Moonlight
- [Moonlight](https://moonlight-stream.org)
  - Open source implementation of NVIDIA's GameStream protocol
:::

```{rst-class} content-collapse
```

(display-socket)=
## Display Socket

  - 3D hardware acceleration
  - Con: Difficult to configure, connect to, and enable GPU hardware acceleration

This is useful for:

- Local Development
  - Enables use of native apps while still isolated from host OS or local environment
  - Applicable for local host development or over high bandwidth and low latency networks
  - Necessary for robust and seamless integration of native apps inside dev containers
- Optimized Performance
  - Enables high frame rates with GPU acceleration using host machine hardware
  - Applicable for visualizing large and dynamic sensor data streams or 3D simulations
  - Necessary for large uncompressed image streams or high frequency signal data

The respective utility for this approach can be summarized as follows:

- Setup Simplicity
  - Although simple in theory, optimal configuration in practice can be fairly involved
  - Limited portability due to reliance of host OS configuration and local prerequisites
- Tooling Flexibility
  - Compatible with most tools and applications across entire ROS ecosystem
  - Suitable when using custom extensions to RViz or RQt plugins
- Robust Integration
  - Simplifies consistent integration between workspace and development tools
  - Suitable when using custom message types or interface definitions
- Responsive UI/UX
  - Native interface and seamless user experience with local display manager
  - Suitable when multitasking between multiple tools and native windows
- GPU Acceleration
  - Requires additional prerequisites and setup to passthrough GPU devices
  - Limited as milage may vary with ease of use being devices vendor specific
- Remote Connectivity
  - Streaming X11 frame buffers via network rather than unix sockets hampers performance
  - Limited to local host development or over high bandwidth and low latency networks
- Bandwidth Efficient
  - Higher bandwidth usage, but can be contingent on frame rate, content, and resolution
  - Limited over networks, although negligible over unix sockets for local development

:::{seealso}
- [x11docker](https://github.com/mviereck/x11docker)
  - Run GUI applications and desktops in docker and podman containers
:::

### Local Host

### Remote Host

https://github.com/microsoft/vscode-remote-release/issues/8031

https://docs.docker.com/config/containers/resource_constraints/#gpu

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html
