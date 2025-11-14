---
title: "Increase the memory heap of the IDE | PyCharm"
source: "https://www.jetbrains.com/help/pycharm/increasing-memory-heap.html"
author:
  - "[[PyCharm Help]]"
published:
created: 2025-11-14
description:
tags:
  - "clippings"
---
## Increase the memory heap of the IDE

Last modified:07 July 2025

The Java Virtual Machine (JVM) running PyCharm allocates some predefined amount of memory. The default value depends on the platform. If you are experiencing slowdowns, you may want to increase the memory heap.

### Increase memory heap

1. Do one of the following:
	- In the main menu, go to Help | Change Memory Settings.
	- Go to Settings | Memory Usage.
2. Set the necessary amount of memory that you want to allocate and click Save and Restart

This action changes the value of the `-Xmx` option used by the JVM to run PyCharm. Restart PyCharm for the new setting to take effect.

PyCharm also warns you if the amount of free heap memory after a garbage collection is less than 5% of the maximum heap size:

![The Low Memory warning](https://resources.jetbrains.com/help/img/idea/2025.2/LowMemoryWarning.png)

The Low Memory warning

Click Configure to increase the amount of memory allocated by the JVM. If you are not sure what would be a good value, use the one suggested by PyCharm.

![The Memory Settings dialog](https://resources.jetbrains.com/help/img/idea/2025.2/py_IncreaseMemoryHeap.png)

The Memory Settings dialog

Click Save and Restart and wait for PyCharm to restart with the new memory heap setting.

### Enable the memory indicator

PyCharm can show you the amount of used memory in the [status bar](https://www.jetbrains.com/help/pycharm/guided-tour-around-the-user-interface.html#status-bar). Use it to judge how much memory to allocate.

To enable the memory indicator, go to Settings | Memory Usage and select the Show Memory Indicator checkbox.

![Memory Usage page opened in Settings](https://resources.jetbrains.com/help/img/idea/2025.2/py_memory_usage.png)

Memory Usage page opened in Settings

## Toolbox App

If you are using the Toolbox App, you can change the maximum allocated heap size for a specific IDE instance without starting it.

1. Open the Toolbox App, click the settings icon next to the relevant IDE instance, and select Settings.
	![Opening IDE instance settings in the Toolbox App](https://resources.jetbrains.com/help/img/idea/2025.2/toolbox-app-open-settings.png)
	Opening IDE instance settings in the Toolbox App
2. On the instance settings tab, expand Configuration and specify the heap size in the Maximum heap size field.

If the IDE instance is currently running, the new settings will take effect only after you restart it.

If you are using a standalone instance not managed by the Toolbox App, and you cannot start it, it is possible to manually change the `-Xmx` option that controls the amount of allocated memory. Create a copy of the default [JVM options](https://www.jetbrains.com/help/pycharm/tuning-the-ide.html#configure-jvm-options) file and change the value of the `-Xmx` option in it.

Memory parameters specific to a particular service, for example, Docker can be modified through the [corresponding run/debug configurations](https://www.jetbrains.com/help/pycharm/run-debug-configuration.html).

Thanks for your feedback!

Was this page helpful?