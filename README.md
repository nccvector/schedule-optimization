
#Roster Generation

Use the google cp-sat solver (https://developers.google.com/optimization/cp) and develop a model similar to that described in their sample model (https://developers.google.com/optimization/scheduling/employee_scheduling) and here - https://github.com/google/or-tools/blob/master/examples/python/shift_scheduling_sat.py

We need to balance hard and soft constraints to determine the best fit of staff to maximise service level (by maximising coverage) against a specified requirement.

## Requirement
The requirement is defined as a staff requirement by skill type in 15 or 30 minute intervals through the (week) roster period.

## Coverage
Coverage is measured as the difference between requirement and allocated staff at each interval in the period.

So for each interval the coverage is measured as:

coverage % = allocated staff /  required staff.

The overall effectiveness of the roster is the % of intervals that fall below 90% coverage. ie if there are 672 intervals in a week, and all except for 30 have > 90% coverage, it would be (672 - 30) / 672 = 96% schedule coverage.

## Shift Patterns (Shift Templates)
Shift patterns can define the start windows, and potential durations for shifts. Shifts have a number of paid hours, and a duration. They are associated with staff members and also workplans.

Shift Templates have hard constraints - min / max numbers of staff that may be assigned to them.

## Staff Members
Staff Members are scheduled based on their contract conditions, skills, and availabilities through the roster period. They also have a have a history that is reflected and used in rotations, and through their association with shift templates (patterns). 

Staff members belong to specified timezone, and their shift patterns and availabilities are always relative to that timezone.

### Staff Member hard constraints
Hard constraints are oriented around user contracts and availabilities and may be summarised as:

The days, hours, and durations that a user may be available through the period.

Minimum and maximum allocated hours through the period

Minimum and maximum hours / shifts per period (week)

Minimum time between shifts

Maximum consecutive working days

The staff member must have the required skill to match the requirement.

## Workplans / Policies
Workplans define the policies related to a set of users and one or more workloads. The policies may comprise of hard and soft constraints, and can be weighted to give a prioritisation to how much they should be considered when determining best fit.

Workplans may be associated with workloads, and if so can specify whether the workplan should ensure that all users are allocated a shift, or only users up to the point that coverage has reached a specified threshold.

### Prioritisation Rules
When determining which staff should be allocated, prioritisation rules can be defined to ensure that staff members with these attributes are allocated by preference.

* employment type  - Permanent / Contractor
* seniority - Prefer more senior staff members first
* tags - prefer staff members with particular tags specified.
* team - allocate to specific teams first.

### Workplan hard constraints
* minimum / maximum staff allocated to specific shift templates.
* Fill to requirement / User availabilities

### Workplan soft constraints
Soft constraints generally cover preferences and rotations. Soft constraints are as follows:

#### Staff Member preferences for workload / start time / rostered day off
If agent preference rules are enabled, any staff member that has a preference defined for the matching workload / start time / rostered day off should be preferred over a similar staff member with no preference defined.

#### Shift Template Rotations
Shift template rotations are based on the users shift history. Shift templates are normally rotated in order - ie if the staff member was on the first template in the work plan last week, they would be rotated to the second one next week. Staff member shift history should be calculated to give fairness over time, and to weight away from the most recent allocations.
* Keep team members together on the same shift template when rotating (ie the whole team must be on the same template). When this rule is applied, the weighting for the shift template to choose should be based on the teams aggregated score.
* Rotate individual staff members to the next template in order based on the last template most allocated through the previous period. This may be defined as a hard constraint or a soft constraint
* Rotate individual users to the next template based on the fairness weighting (from history)
* Rotate all users to the next template based on the aggregate majority allocation across all users in the previous period.

#### Workload rotations
Workload rotations are similar to shift template rotations in that they use historical shifts to determine what workloads individual staff members have worked on, and attempting to rotate them to new workloads based on a fair allocation, and to weight away from the most recent allocations.

#### Shift Start Time Fairness
Use a weighting of historical shift patterns to determine a start time for a shift (where the allocated shift template has a flexible start / end time window). This should be on a fairness basis, and weighted away from the most recent allocations.

#### Start Time Rotation
The period after which the shift start time may be changed. For example if the start time rotation period is set to 1 week, the shift start times for the same user should be kept consistent through the week.

#### Non-working days (RDO)
When a staff member has flexibility in working days (eg may work any 5 out of 7 days, non-working days policies may be applied.

* The maximum number of weekends worked in a specified period. The shift history may be used to determine the previous working days.
* Prefer non-working days to be consecutive
* Mandatory working days - if this rule is specified all staff members in the work plan must be allocated

## Inputs
The inputs to the optimisation process are:
* A list of workplans defining the users / policies to be applied
* A set of requirements (ie required staff by interval, by workload)
* A period start and end

## Outputs

The outputs are a json document that contains the following attributes:
* An array of shifts with start / end times, associated workload, and the associated users
* The overall coverage
* A list of any hard constraints that were unable to be met

## Scenarios
There are a number of test scenarios defined that outline this behaviour. See the readme in each of the README files under /test/scenariox