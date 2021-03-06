**_Antares\_Simulator 7.1.0_**

**_OPTIMIZATION PROBLEMS_**

**_FORMULATION_**

**Document available on : [https://antares-simulator.org](https://antares-simulator.org/)**

| **Simulation package** | **X** |
| --- | --- |
| **Script Editor package** |
 |
| **Graph Editor package** |
 |
| **Data Organizer package** |
 |

**_OPTIMIZATION PROBLEMS_**

**_FORMULATION_**

##### Table of contents

#

[**1 Introduction** 3](#_Toc529966263)

[**2 Typology of the problems solved** 4](#_Toc529966264)

[**3 Notations** 6](#_Toc529966265)

[**4 Formulation of problem** 10](#_Toc529966266)

[**4.1 Objective** 10](#_Toc529966267)

[**4.1 Constraints related to the nominal system state** 10](#_Toc529966268)

[**4.2 Constraints related to the uplifted system state (activation of security reserves)** 12](#_Toc529966269)

[**5 Formulation of problem** 12](#_Toc529966270)

[**5.1 Objective** 12](#_Toc529966271)

[**5.2 Constraints** 12](#_Toc529966272)

[**6 Antares as a SCOPF (&quot;FB model&quot;)** 13](#_Toc529966273)

[**7 Antares as a SCOPF (&quot;KL model&quot;)** 13](#_Toc529966274)

[**7.1 Implementation of Kirchhoff&#39;s second law** 13](#_Toc529966275)

[**7.2 Implementation of a passive loop flow** 13](#_Toc529966276)

[**7.3 Modelling of phase-shifting transformers** 14](#_Toc529966277)

[**7.4 Modelling of DC components** 14](#_Toc529966278)

[**7.5 Implementation of security rules N-1,..., N-k** 14](#_Toc529966279)

[**8 Antares as a SCOPF (&quot;KL+FB model&quot;)** 15](#_Toc529966280)

[**9 Miscellaneous** 15](#_Toc529966281)

[**9.1 Modelling of generation investments in** 15](#_Toc529966282)

[**9.2 Modelling of pumped storage power plants** 15](#_Toc529966283)

#### **1 Introduction**

The purpose of this document is to give every user of the _ **Antares\_Simulator** _ model (regardless of its version number
# 1
), a detailed and comprehensive formulation of the main optimization problems solved by the application&#39;s inner optimization engine.

The aim of the information presented hereafter is to provide a transparent access to the inner workings of the software from a **formal** standpoint. Note that, aside from this conceptual transparency, the software itself offers an option that makes it possible for the user to print, in a standardized format, any or all of the optimization problems actually solved in the course of an _ **Antares\_Simulator** _ session
# 2
. Used together with the elements developed in the next pages, this **practical** access to the internal model implemented the tool allows fair and open benchmarking with comparable software. Besides, another important issue regarding transparency is addressed by the release of _ **Antares\_Simulator** _ as an Open Source Gnu GPL 3.0 application.

So as to delimit the scope of the present document with as much clarity as possible, it is important to notice that a typical _ **Antares\_Simulator** _ session involves different steps that are usually run in sequence, either automatically or with some degree of man-in-the-loop control, depending on the kind of study to perform.

These steps most often involve:

1. GUI session dedicated to the initialization or to the updating of various input data sections (load time-series, grid topology, wind speed probability distribution, etc.)
2. GUI session dedicated to the definition of simulation contexts (definition of the number and consistency of the &quot;Monte-Carlo years&quot; to simulate)
3. Simulation session producing actual numeric scenarios following the directives defined in (b)
4. Optimization session aiming at solving all of the optimization problems associated with each of the scenarios produced in (c).
5. GUI session dedicated to the exploitation of the detailed results yielded by (d)

The scope of this document is exclusively devoted to step (d). Note that equivalent information regarding the other steps may be found in other documents made available either:

- Within the application itself, in the &quot; ?&quot; menu
- On the _ **Antares\_Simulator** _
# 3
 website (download section) : [https://antares-simulator.org](https://antares-simulator.org/)
- In technical publications referenced in the bibliography section of the website

The following picture gives a functional view of all that is involved in steps (a) to (e). In this illustration, Step (d), whose goal is to solve the problems introduced in this document, is materialized by the red box.

![](RackMultipart20201119-4-hpmtl9_html_ca6164c8f4d9c041.gif)

The number and the size of the individual problems to solve (a least-cost hydro-thermal unit-commitment and power schedule, with an hourly resolution and throughout a week, over a large interconnected system) make optimization sessions often computer-intensive. Note that the content of the blue &quot;hydro energy manager&quot; box appearing on the previous figure, whose purpose is to deal with energy storage issues at the seasonal scale, is not detailed in the present document but in the &quot;General Reference Guide&quot;.

Depending on user-defined results accuracy requirements, various practical options allow to simplify either the formulation of the weekly UC &amp; dispatch problems (e.g. do not account for constraints associated with operational reserves) or their resolution (i.e. find, for the native MILP, an approximate solution based on two successive LPs). For the sake of simplicity and clarity, the way these options are used to revise the primary problem formulation is not detailed hereafter. Likewise, many simplifications are introduced to keep notations as light as possible. This is why, for instance, the overall sum of load, wind power generation, solar power generation, run of the river generation, and all other kinds of so-called &quot;must-run&quot; generation is simply denoted &quot;load&quot; in the present document.

#### **2 Typology of the problems solved**

In terms of power studies, the different fields of application Antares has been designed for are the following:

- **Generation adequacy problems :** assessment of the need for new generating plants so as to keep the security of supply above a given critical threshold

What is most important in these studies is to survey a great number of scenarios that represent well enough the random factors that may affect the balance between load and generation. Economic parameters do not play as much a critical role as they do in the other kinds of studies, since the stakes are mainly to know if and when supply security is likely to be jeopardized (detailed costs incurred in more ordinary conditions are of comparatively lower importance). In these studies, the default Antares option to use is the &quot;Adequacy&quot; simulation mode, or the &quot;Draft&quot; simulation mode (which is extremely fast but which produces crude results).

- **Transmission project profitability :** assessment of the savings brought by a specific reinforcement of the grid, in terms of decrease of the overall system generation cost (using an assumption of fair and perfect market) and/or improvement of the security of supply (reduction of the loss-of-load expectation).

In these studies, economic parameters and the physical modeling of the dynamic constraints bearing on the generating units are of paramount importance. Though a thorough survey of many &quot;Monte-Carlo years&quot; is still required, the number of scenarios to simulate is not as large as in generation adequacy studies. In these studies, the default Antares option to use is the &quot;Economy&quot; simulation mode.

- **Generation and/or Transmission expansion planning:** rough assessment of the location and consistency of profitable reinforcements of the generating fleet and/or of the grid at a given horizon, on the basis of relevant reference costs and taking into account feasibility local constraints (bounds on the capacity of realistic reinforcements).

These long term studies clearly differ from the previous ones in the sense that the generation and transmission assets that define the consistency of the power system are no longer passive parameters but are given the status of active problem variables. In the light of both the nature and the magnitude of the economic stakes, there is comparatively a lesser need than before for an accurate physical modeling of fine operational constraints or for an intensive exploration of a great many random scenarios. The computer intensiveness of expansion studies is, however, much higher than that of the other types because the generic optimization problem to address happens to be much larger.

The common rationale of the modeling used in all of these studies is, whenever it is possible, to decompose the general issue (representation of the system behavior throughout many years, with a time step of one hour) into a series of standardized smaller problems.

In _ **Antares\_Simulator** _, the &quot;elementary &quot; optimization problem resulting from this approach is that of the minimization of the overall system operation cost over a week, taking into account all proportional and non-proportional generation costs, as well as transmission charges and &quot;external&quot; costs such as that of the unsupplied energy (generation shortage) or, conversely, that of the spilled energy (generation excess).

In this light, carrying out generation adequacy studies or transmission projects studies means formulating and solving a series of a great many week-long operation problems (one for each week of each Monte-Carlo year ), assumed to be independent. This generic optimization problem will be further denoted **,** where is an index encompassing all weeks of all Monte-Carlo years.

Note that this independency assumption may sometimes be too lax, because in many contexts weekly problems are actually coupled to some degree, as a result of energy constraints (management of reservoir-type hydro resources, refueling of nuclear power plants, etc.). When appropriate, these effects are therefore dealt with before the actual decomposition in weekly problems takes place, this being done
# 4
 either of the following way (depending on simulation options):

1. Use of an economic signal (typically, a shadow &quot;water value&quot;) yielded by an external preliminary stochastic dynamic programming optimization of the use of energy-constrained resources.
2. Use of heuristics that provide an assessment of the relevant energy credits that should be used for each period, fitted so as to accommodate with sufficient versatility different operational rules.

Quite different is the situation that prevails in expansion studies, in which weekly problems cannot at all be separated from a formal standpoint, because new assets should be paid for all year-long, regardless of the fact that they are used or not during such or such week : the generic expansion problem encompasses therefore all the weeks of all the Monte-Carlo years at the same time. It will be further denoted

The next sections of this document develop the following subjects:

- Notations used for and

- Formulation of

- Formulation of

- Complements to the standard problems (how to make _ **Antares\_Simulator** _ work as a SCOPF )

- Miscellaneous complements to the standard problems

#### **3 Notations**

####

#### **4 Formulation of problem**

Superscript k is implicit in all subsequent notations of this section (omitted for simplicity&#39;s sake)

##### **4.1 Objective**

##### **4.1 Constraints related to the nominal system state**

Balance between load and generation:

First Kirchhoff&#39;s law:

On each node, the unsupplied power is bounded by the net positive demand:

On each node, the spilled power is bounded by the overall generation of the node (must-run + dispatchable power):

Flows on the grid:

Flows are bounded by the sum of an initial capacity and of a complement brought by investment

Binding constraints :

Reservoir-type Hydro power:

The energy generated throughout the optimization period is bounded

Instantaneous generating power is bounded

Intra-daily power modulations are bounded

Instantaneous pumping power is bounded

Reservoir level evolution depends on generating power, pumping power, pumping efficiency, natural inflows and overflows

Reservoir level is bounded by admissible lower and upper bounds (rule curves)

Thermal units :

Power output is bounded by must-run commitments and power availability

The number of running units is bounded

Power output remains within limits set by minimum stable power and maximum capacity thresholds

Minimum running and not-running durations contribute to the unit-commitment plan. Note that this modeling requires
# 5
 that one at least of the following conditions is met:

##### **4.2 Constraints related to the uplifted system state (activation of security reserves)**

All constraints to previously defined for regular operation conditions are repeated with replacement of all variables by their twins when they exist.

Besides, in the expression of constraints , all occurrences of are replaced by

#### **5 Formulation of problem**

##### **5.1 Objective**

##### **5.2 Constraints**

#### **6 Antares as a SCOPF (&quot;FB model&quot;)**

When problems and do not include any instance of so-called &quot;binding constraints&quot; and if no market pools are defined, the flows within the grid are only committed to meet the bounds set on the initial transmission capacities, potentially reinforced by investments (problem ).In other words, there are no electrical laws enforcing any particular pattern on the flows, even though hurdles costs and may influence flow directions through an economic signal.

In the general case, such a raw backbone model is a very simplified representation of a real power system whose topology and consistency are much more complex. While the full detailed modeling of the system within Antares is most often out of the question, it may happen that additional data and/or observations can be incorporated in the problems solved by the software.

In a particularly favorable case, various upstream studies, taking account the detailed system characteristics in different operation conditions (generating units outages and/or grid components outages N, N-1 , N-k,…) may prove able to provide a translation of all relevant system limits as a set of additional linear constraints on the power flowing on the graph handled by Antares.

These can therefore be readily translated as &quot;hourly binding constraints&quot;, without any loss of information. This kind of model will be further referred to as a &quot;FB model&quot;
# 6
. Its potential downside is the fact that data may prove to be volatile in short-term studies and difficult to assess in long-term studies.

####

#### **7 Antares as a SCOPF (&quot;KL model&quot;)**

When a full FB model cannot be set up (lack of robust data for the relevant horizon), it remains possible that classical power system studies carried on the detailed system yield sufficient information to enrich the raw backbone model. An occurrence of particular interest is when these studies show that the physics of the active power flow within the real system can be valuably approached by considering that the edges of behave as simple impedances .This model can be further improved if a residual (passive) loop flow is to be expected on the real system when all nodes have a zero net import and export balance (situation typically encountered when individual nodes actually represent large regions of the real system). This passive loop flow should therefore be added to the classical flow dictated by Kirchhoff&#39;s rules on the basis of impedances . This model will be further referred to as a &quot;KL model&quot;
# 7
. Different categories of binding constraints, presented hereafter, make it possible to implement this feature in and

##### **7.1 Implementation of Kirchhoff&#39;s second law**

The implementation ofKirchhoff&#39;s second law for the reference state calls for the following additional hourly binding constraints:

##### **7.2 Implementation of a passive loop flow**

In cases where a residual passive loop flow should be incorporated in the model to complete the enforcement of regular Kirchhoff&#39;s rules, the binding constraints mentioned in 7.1 should be replaced by:

##### **7.3 Modelling of phase-shifting transformers**

In cases where the power system is equipped with phase-shifting transformers whose ratings are known, ad hoc classical power studies can be carried out to identify the minimum and maximum flow deviations and phase-shift that each component may induce on the grid. The following additional notations are in order:

The enhancement of the model with a representation of the phase-shifting components of the real system then requires to re-formulate as follows the binding constraints defined in 7.2:

##### **7.4 Modelling of DC components**

#####

When the power system graph contains edges that represent DC components, additional notations need be defined:

The proper modeling of the system then requires that all constraints identified in 7.1, 7.2, 7.3 be formulated using notations instead of

##### **7.5 Implementation of security rules N-1,..., N-k**

Upstream power system classical calculations on the detailed system are assumed to have provided appropriate estimates for line outage distribution factors (LODFs) for all components involved in the contingency situations to consider. The following additional notations will be further used:

The implementation of security rules for the chosen situations requires the following additional binding constraints:

#### **8 Antares as a SCOPF (&quot;KL+FB model&quot;)**

If the information context is rich enough, it is possible to set up a hybrid model based on both previous approaches: on the one hand, a &quot;KL layer&quot; makes use of the best available estimates for grid impedances and loop flows, so as to instantiate physically plausible flow patterns; on the other hand, a &quot;FB layer&quot; sets multiple kinds of limits on the admissible flow amplitudes, as a result of various operation commitments.

To work appropriately, such a hybrid model needs an additional auxiliary layer that performs a mapping between the two &quot;twin&quot; FB and KL grid layers
# 8
.

#### **9 Miscellaneous**

##### **9.1 Modelling of generation investments in**

The assessment of the desired level of expansion of any segment of the generating fleet can be carried out with a model in which the potential reinforcements of the fleet are assumed to be actually commissioned from the start but located on virtual nodes connected to the real system through virtual lines with a zero initial capacity. Relevant generation assets costs and capacities should then be assigned to the virtual connections, and the investment in new generation can be optimized &quot;by proxy&quot; through the identification of the optimal expansion of the virtual connections.

##### **9.2 Modelling of pumped storage power plants**

A number of specific equipments, as well as particular operation rules or commercial agreements, can be modelled with appropriate use of binding constraints. A typical case is that of a pumped storage power plant operated with a daily or weekly cycle. Such a component can be modelled as a set of two virtual nodes connected to the real grid through one-way lines. On one node is attached a virtual load with zero VOLL, which may absorb power from the system. On the other node is installed a virtual generating unit with zero operation cost, which may send power to the system. The flows on the two virtual lines are bound together by a daily or weekly constraint (depending on the PSP cycle duration), with a weight set to fit the PSP efficiency ratio. Besides, time offsets may be included in the constraints to take into account considerations regarding the volume of the PSP reservoir (additional energy constraint).

[1](#sdfootnote1anc) The development of the product is a continuous process resulting in the dissemination of a new version each year. As a rule, version N brings various improvements on the code implemented in version N-1 and enhances the functional perimeter of the tool. This document presents the general optimization problem formulation as it is formalized so far in the last version of disseminated version of _ **Antares\_Simulator** _ (V7).

[2](#sdfootnote2anc) Reference guide , section « optimization preferences : &quot;export mps problems&quot;

[3](#sdfootnote3anc) For the sake of simplicity, the _ **Antares\_Simulator** _ application will be further denoted « Antares »

[4](#sdfootnote4anc)See «hydro» sections of the General Reference Guide (&quot;hydro&quot; standing as a generic name for all types of energy storage facilities)

[5](#sdfootnote5anc)This does not actually limit the model&#39;s field of application: all datasets can easily be put in a format that meets this commitment

[6](#sdfootnote6anc) FB stands for « flow-based », denomination used in the framework given to the internal electricity market of western Europe

[7](#sdfootnote7anc) KL stands for &quot;Kirchhoff- and-Loop&quot;. Such a model was used in the European E-Highway project ([http://www.e-highway2050.eu](http://www.e-highway2050.eu/))

[8](#sdfootnote8anc) A common situation is that KL and FB are defined at different spatial scales and describe different topologies: the KL layer has typically a fairly large number of small-sized regions, while the FB layer consists of fewer wide market zones. The role of the auxiliary layer is to implement the appropriate relationship between physical regions and trade zones.

Copyright © RTE 2007-2019 – Version 7.1.0

Last Rev : M. Doquet - 16 OCT 2019

30/30