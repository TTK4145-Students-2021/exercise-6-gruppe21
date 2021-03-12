
# Detailed Synchronizer module design

Our design philosophy is to have minimal fault-handling and simple module design, which we aim to achieve by having an order synchronizer module which handles the "difficult"
task of synchronizing and delegating hall orders between the different elevators. This module should be able to handle most failures, including elevators disconnecting, motors losing power and programs crashing. It should also be able to automatically get an elevator up to speed with the current state of the system after restart/reconnect.

The main part of the module will consist of a floor-order state machine, and functions to handle hall-order state events. Each elevator will have a state for each of the hall call buttons,
with four different possible state values. All elevators will broadcast their floor-order state machines, and the different elevators communicate by comparing their floor-order states. To make the system robust, the ways in which a floor-order state can change is strict.<br>
## Synchronizer FSM
<img width="614" alt="OrderFSMfigure" src="https://user-images.githubusercontent.com/61008623/110940089-a5ea4b00-8336-11eb-81e3-7a7c611ab66f.png">

- Table with one entry for each hall button type. Four possible states: None - New - Handling - Complete
- The synchronizer state is broadcasted by all elevators. 
- Transitions from left to right, except when an order changes from Complete to None.<br>

None -> New
- A button is pressed
- Another syncFSM is in the New or Handling state

New -> Handling
- The elevator with lowest cost function takes the order and turn on the light.
- order time-to-complete exceeded

Handling -> Complete
- The transition from Handling to Complete occurs when the door opens at the desired floor. The order will be in the Complete state for ~100 milliseconds

New -> Complete
- The syncFSM see that another elevator has completed the order

Complete -> None
- The transition happens after x ms. This should be enough time for all elevators to change to Complete.

# Error handling
If an elevator does not have the lowest cost function, it remains in the New state and starts a timer. If the order is not completed before the timeout, another elevator takes the order. This ensures that the orders will be served if an elevator disconnects, is obstructed, etc.
The syncFSM is initialized with all orders as None. An elevator is initialized when it reconnects after being disconnected. As the None state has no impact on the other syncFSMs, elevators disconnecting and reconnecting will not affect on the existing orders.

