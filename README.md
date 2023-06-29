# Explaining A Staff Rostering Problem By Mining Trajectory Variance Structures - SGAI 2023

## Datasets
Collection of data files used to create the problem instances specified in the SGAI 2023 paper - Explaining A Staff Rostering Problem By Mining Trajectory Variance Structures DOI:Pending.

Data file descriptions:
* **data/employeeData.csv** - This file outlines the roster preferences for each Employee. Mapped to the problem in this paper, this file represents the variable Roster-Week data shown in Table1.a of the paper.
 *   **data/rosterPatternData.csv** - This file contains the total number of Employees currently assigned to each roster pattern at the start of the problem initialization. This file also outlines the number of weeks that each roster pattern lasts for. This is used, in conjunction with **employeeData.csv**, to create the Roster-Week tables for each emplyoyee. Each Employee has between 1 and 5 possible rosters and each roster has a set length in weeks. The combination of these creates the Roster-Week table for each Employee.
*    **data/rosterPatternDaysData.csv** - This file outlines the hours to be worked on each day of the week, according to the roster pattern selected. Each roster pattern has a set starting and ending time for each day of the week and is used to calculate whether an Employee is scheduled to work on any given day. This is then used in the Range calculations as part of the fitness function.


## Problem Definition

The optimization problem outlined is a Staff Rostering problem in which the working hours of each Employee in a workforce are optimized for a consistant level of resource availability throughout the week, over a 12 week period of time. In this instance there are:

* 141 Employees
* 100 rosters of varing lengths
  * Each roster specifies working hours
* 1 to 5 possible rosters, pre-allocated to each Employee
* A "Starting State" representing the current working hours of all Employees
* Fitness function designed to minimize "Range" between active Employee counts on each day of the week

The objective function of the problem instance used in this paper is shown here:

$\text{minimize:}    \sum_{1\leq d\leq7}^{}w_d R_d^2 + \left(\sum_{n \in x} P_n\right)S$

$R_d = \frac{\underset{d}{max}\left( a_{kd} \right) - \underset{d}{min}\left( a_{kd} \right)}{\underset{d}{max}\left( a_{kd} \right)}$

$\text{subject to:}  \sum_{i=1}^{n} \text{CV}_i \leq 0.2 \cdot \text{len}(x)$



"The cost function shown in the above function aims to minimise the overall range between the number of workers assigned to work on each of the week days. This calculation has a set of weights, $w$, applied to each day of the week to reduce the impact of the lower availability of workers during the weekend. These are applied to each day $d$, with the values being set at $w=\left( 1,1,1,1,1,10,10\right)$. This was done as  "..a range of 10 on Saturday should not be considered the same as a range of 10 on any other day of the week due to the smaller number of attending resources".
The function contains constraints designed to minimize the disruption to the workforce. The hard constraint, shown in the Equation, defines $CV$ as the total number of workers who have been assigned to a new roster from their sub-pool. This constraint aims to limit the total number of workers from being assigned new rosters to 20\% of the total workforce.

The cost function contains a soft constraint linked to the second summand of $x$, the set of all variables in a solution, and $P = (p_{n})$, a binary array in which $p_{n} = 1$ if the value of variable $n$ results in two consecutive Saturdays being scheduled or $0$ otherwise, due to changing from the initial Roster pattern and week to those outlined in a solution. This soft constraint adds a small penalty of $S = 0.01$ for each violation of this constraint in each solution to help reduce the total occurrences of this. As the aim is to reduce the total range to 0, a minimum value of 0 would be achieved should all ranges be reduced to 0 and no consecutive Saturdays be worked."

The constraint value of $CV$ is the number of individuals of the total workforce that has been assigned a roster other than their current allocation. In this problem instance, as there are 141 Employees, this would result in a solution being considered infeasible should 29 or more Employees be moved from their currently assigned roster set out in the Starting State, to another roster. Moving to any given week in their Starting State assigned roster however does not count as a penalty as all Employees can be moved within their current roster with minimal workforce disruption.

## Starting State

As noted, all optimisation runs use the same starting point. This represents the current state of the optimization problem that the end-user is attempting to improve upon by running the GA. The following is this starting state allocation of rosters to workers:

    #Initial Solution:
    36,24,2,0,10,10,58,3,11,16,38,72,7,9,0,9,19,46,6,20,4,61,8,10,18,3,31,6,60,64,6,8,10,13,0,0,11,29,10,11,40,35,8,38,27,57,6,51,11,49,18,2,1,10,16,5,14,12,11,29,3,30,26,57,37,25,43,29,4,24,27,10,6,10,36,43,0,34,1,9,6,3,5,9,7,3,46,7,31,2,17,6,0,37,18,25,0,23,9,10,5,39,26,8,0,25,12,37,4,62,9,9,58,5,26,8,54,30,42,34,10,21,1,2,3,10,6,38,29,5,3,60,9,12,21,21,8,29,11,38,25



## Example Range Calculation

From the inital solution, the Employee specific Roster-Week table is generated as such for Employee 0 as an example:


Employee 0 roster preferences (from **employeeData.csv**) are:

Preference 1  | Preference 2 | Preference 3 | Preference 4 | Preference 5
------------- | ------------- | ------------- | ------------- | -------------
65            | 55           | 17           | 56            | 80


For each preference, the length of each roster is taken from **rosterPatternData.csv**. Together this generates the Roster-Week table, of which an extract for Employee 0 is shown below. In bold is the Roster-Week allocation currently selected in the Initial Solution Starting State. Employee 0, in the problem as variable $x_1$ has a value of 36. This value, $x_1$'s Roster-Week table equates to Roster 17, Week 10. 

Index | Roster | Week
----- |------- |------ 
1   |   65  | 1
2   |   65  | 2
3   |   65  | 3
...   |  ...  | ....
13   |   65  | 13
14   |   55  | 1
15   |   55  | 2
... | ... | ...
35   |   17  | 9
**36**   |   **17**  | **10**
37   |   17  | 11
... | ... | ...

This will be the new starting Roster and Week number for that Employee. They will then work this roster for the next 12 weeks. Should the roster end before this - as is the case with this example, then the Employee will start roster 17 from week 1 again. The will work the hours specified by roster 17, starting at week 10, then 11, 12, 13, 1, 2 and so forth for a 12 week period. The working hours are specified in the **rosterPatternDaysData.csv** file. An example is shown here for roster 17, starting at week 10:

| Roster | Week | Day | Start | End | Working |
|--------|------|-----|-------|-----|---------|
| 17     | 10   | Mon | 800   | 1740| Yes     |
| 17     | 10   | Tue | 800   | 1740| Yes     |
| 17     | 10   | Wed | 800   | 1740| Yes     |
| 17     | 10   | Thu | 0     | 0   | No      |
| 17     | 10   | Fri | 800   | 1740| Yes     |
| 17     | 10   | Sat | 0     | 0   | No      |
| 17     | 10   | Sun | 0     | 0   | No      |
| 17     | 11   | Mon | 0     | 0   | No      |
| 17     | 11   | Tue | 0     | 0   | No      |
| 17     | 11   | Wed | 800   | 1740| Yes     |
| 17     | 11   | Thu | 800   | 1740| Yes     |
| 17     | 11   | Fri | 800   | 1740| Yes     |
| 17     | 11   | Sat | 800   | 1740| Yes     |
| 17     | 11   | Sun | 0     | 0   | No      |
| 17     | ...  | ... | ...   | ... | ...     |

For each variable in a solution, and so for each of the 141 Employees represented by a solution, similar tables are generated. This means that for each Employee we check the solution for their assigned roster and starting week. We then calculate whether they are working or off for each day in the 12 week period. Once complete, we sum the number of workers assigned to work for each day and use this in the Range calcualtion section of the fitness function. As specified in the paper - $\left(\sum_{n \in x} P_n\right)S$ adds a penalty to the fitness value for each Employee who works two consecutive saturdays once the new rosters have been allocated. In our example, the $CV$ value would be increased by 1 for any given solution $X$, if $(x_1 < 26)$ or $(x_1 > 39)$ as a value less than 26 or more than 39 would indicate that the solution has assigned Employee 0 a different roster than the starting state of the problem.

