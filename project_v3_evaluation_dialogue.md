---
name: Simulator v3 Evaluation Dialogue — June 4 2026
description: Verbatim user evaluation of v3 simulator — 20+ questions on UI, physics, MC, optimizer, archetypes. For EOD HTML post-script.
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---

## VERBATIM USER EVALUATION (keep for EOD HTML)

"Now I will go through the v3. Pls tell me what you changed in a table format. With equations, reasoings and expected fix for each. These will be in a post-script so keep in memory for EOD HTML.

Good keep these summaries verbatim to link list in EOD HTML for the v3 bug_fixes and reasonsings. Now for the evaluation. What is the Full_Config button? It does not do anything. What is the stable button? Why does the reset button set to what? The aggregate button along the 2.4, 5 6 ... how do you aggregate? What inputs do you get from the user which says what bands the AP is on? How is the Aggregate calculated? In MonteCarlo, why is it hard coded to P50/ P75 or P99, can you not take the user-input here and then run the MonteCarlo? In the Gap ANalysis, how is POS latency calculated? What triggers it in the user input? There seems to be 20+ parameters in the GapAnalyses? For a reason. We want to broaden the 'horizon' of this simulator into the other 'archetypes' ... so think of ways we can 'scale this effort' to those areas, where the 45 parameters that are 'frozen' may not be the same 45 parameters that are here in the Stadium. So, we will have a selectable choice for the user on 'which archetype he/ she is looking to study'? What happens when the user chooses Target %ile MC+Optimizar? This is where we can give the user the standard chices or if he/ she chooses her own let them give you that number to run with? The AP Array, where do the numbers come from, the number of APs, like 100 in this picture? In the service class SLA below, I cannot read the fonts clearly? POS FIAL VIP TEMs PASS then Guest Fail. I need to be able to click in each category and understand what factors contributed to the failur, separatey from the overall GapAnalyses and have the system suggest what could be changed to improve it? On the right hand side below the array, Regarding the SLA breach, I want to undestand clearly where the user can input the SLA parameters. The breaking point finter I dont clearly understand. There is concurrency and airtime and it says find the threhsold? Explain. The 'Full Config' button, what does it do? Cannot seem to understand. Below the SLA line with inputs... this section needs to be scrollable... cannot see iside have to go down/ up arrow. Running monte-carlo, we should have a small statment on what it does? I can't connect the demand to the Pareto, can you explain those two new knobs? The rounds for optimizer knobs, how is it affecting the output? I am looking to 'tune the knob' to get to my confidence factor with the 12-parameters set to produce the QoE/ SLA. This was my holy grail. When it says apply to simulator, what exactly is it doing? Check robustness too? Applying to simulator actually iproves peformance and makes it align to SLA. So, we need to clearly tell the user? KEEP THIS DIALOGUE VERBATIM FOR EOD_UPDATE so we know how we arrived at the next fork. Now, GI is 1600ns, we can go below than 800ns? Other items next. Pls review comments above."

## ISSUES LOG (from evaluation — for Sprint 2 planning)

1. Full Config modal: may not be opening visibly — needs fix/test
2. Stable badge: static, should be dynamic based on score
3. Reset: resets to applyOptimal() values, not factory defaults — needs label clarification
4. MC percentile: hardcoded 3 options — add free-input field for custom %ile
5. SLA fonts: too small — Sprint 2 font audit
6. SLA cards: not clickable — add drill-down per service class showing contributing factors + suggestions
7. SLA thresholds: hardcoded in JS — expose as user-editable inputs
8. Panel below SLA inputs: not scrollable — CSS fix needed
9. Monte Carlo: no description text — add 1-line explainer
10. Pareto knobs: not explained in UI — add tooltip/description
11. "Apply to Simulator" button: needs clearer label explaining what it does
12. "Check Robustness": needs clearer label
13. Archetype selector: new feature — selectable venue type changes frozen/search param sets
14. Target %ile: add free-text input alongside dropdown
15. GI: 800ns IS the minimum for 802.11ax/be — simulator is correct, no fix needed

## ARCHITECTURAL DECISIONS FROM EVALUATION

- Archetype system: each archetype defines its own frozen/search param split + SLA targets + default values
- SLA thresholds should be user-editable inputs, not hardcoded
- MC percentile should accept user-defined value (not just P50/P75/P90)
- "Apply to Simulator" = set all 12 search knobs to optimizer's best values → triggers update()
- "Check Robustness" = run 500-iteration MC on current settings, report P95 vs SLA threshold
