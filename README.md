# Logistic Task Language

Schedular for LoTLan tasks. Parses LoTLan files and accepts events defined in the files to schedule.



## Task examples

You can find many LoTLan files in the [examples folder](./etc/examples). For instance
[a simple task](./etc/examples/Scheduling/001_simple_task.tl), [a task with a triggeredBy and finishedBy](./etc/examples/Scheduling/004_trb_fb.tl) and [a task with an onDone](./etc/examples/Scheduling/013_on_done.tl) and many more.

## Quickstart / Example

Here is an example how the schedular can be used
```python
import sys

from lotlan_schedular.schedular import LotlanSchedular
from lotlan_schedular.api.event import Event


def cb_triggered_by(mf_uuid, _uuid, event_information):
    print("cb_triggered_by from mf: " + str(mf_uuid))
    print("UUID: " + str(_uuid), "Event_Info: " + str(event_information))
    # foreach event in event_information


def cb_next_to(mf_uuid, transport_orders):
    print("cb_next_to from mf: " + str(mf_uuid))
    print(str(transport_orders))


def cb_finished_by(mf_uuid, _uuid, event_information):
    print("cb_finished_by from mf: " + str(mf_uuid))
    print("UUID: " + str(_uuid), "Event_Info: " + str(event_information))


def cb_task_finished(mf_uuid, _uuid):
    print("cb_task_finished from mf: " + str(mf_uuid))
    print("task with uuid " + str(_uuid) + " finished")


def cb_all_finished(mf_uuid):
    print("cb_all_finished from mf: " + str(mf_uuid))


def main():
    test_flag = False
    lotlan_string = ""

    if len(sys.argv) >= 2:
        if sys.argv[1] == "--test":
            test_flag = True
            with open(sys.argv[2], 'r') as file:
                lotlan_string = file.read()
        else:
            with open(sys.argv[1], 'r') as file:
                lotlan_string = file.read()

        lotlan_logic = LotlanSchedular(lotlan_string, test_flag)
        material_flows = lotlan_logic.get_materialflows()

        for material_flow in material_flows:
            material_flow.register_callback_triggered_by(cb_triggered_by)
            material_flow.register_callback_next_to(cb_next_to)
            material_flow.register_callback_finished_by(cb_finished_by)
            material_flow.register_callback_task_finished(cb_task_finished)
            material_flow.register_callback_all_finished(cb_all_finished)
            material_flow.start()

        material_flow_running = True
        while (material_flow_running):
            _input = str(input("Wait for input:>"))
            mf_number = 0
            uid = 0
            input_name = "buttonPressed"
            input_value = "True"

            if _input != "":
                mf_number, uid, input_name, input_value = _input.split(" ")

            mf_number = int(mf_number)

            if mf_number < len(material_flows):
                material_flows[mf_number].fire_event(str(uid), Event(input_name, "", "bool", input_value == "True"))

            # check if a material flow is still running
            # if every material flow is finished we are done otherwise continue
            material_flow_running = False
            for mf in material_flows:
                if mf.is_running() is True:
                    material_flow_running = True

if __name__ == '__main__':
    main()

```

## License
Copyright [2020] [Fraunhofer IML]

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## Contributors
Peter Detzner, Maximilian Hoerstrup, Dominik Lux


## Conference

P. Detzner, T. Kirks and J. Jost, "[A Novel Task Language for Natural Interaction in Human-Robot Systems for Warehouse Logistics](https://ieeexplore.ieee.org/document/8845336)", 2019 14th International Conference on Computer Science & Education (ICCSE), Toronto, ON, Canada, 2019, pp. 725-730.