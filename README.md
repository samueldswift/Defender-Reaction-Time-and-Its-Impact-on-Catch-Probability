# Defender-Reaction-Time-and-Its-Impact-on-Catch-Probability
Quantifying the twitch of NFL cornerbacks and linking it to a success metric like "expected completion percentage".


                                                        2026 NFL Big Data Bowl - Analytics
                                                        
                            Quantifying Quick Twitch Coverage: Defender Reaction Time and Its Impact on Catch Probability
                                                        
                                                                17 December 2025
                                                        
                                                                  By: Sam Swift


**Executive Summary**

This project builds a metric called defender reaction time (reaction_time), which measures how quickly the nearest coverage defender “triggers” on a pass. A defender is marked as reacting when two things happen at once: their distance to the targeted receiver shrinks for several straight frames (about 0.4 seconds at 10 Hz), and their acceleration along the defender‑to‑receiver line exceeds a data‑driven threshold. Together, these conditions filter out noise and focus on sustained closing toward the receiver.
Using tracking and play‑by‑play data, I first build a catch‑probability model at ball arrival using only separation, ball and receiver location, and ball flight time. Logistic regression, decision tree, random forest, and XGBoost models are fit; the best model’s probabilities are merged back to each play as expected catch chance. I then add ‘reaction_time’ as a feature in a new version of this model. In an L1‑penalized logistic regression on standardized features, ‘reaction_time’ remains a significant predictor: a one‑standard‑deviation decrease in ‘reaction_time’ (earlier trigger) makes a completion less likely, even after accounting for separation and ball placement.
With the link between faster ‘reaction_time’ and lower completion probability established, the analysis shifts to the on‑field situations that tend to produce quick reactions. Defensive coordinators who want to cut catch probability can lean into coverages and roles that let their players trigger early. I summarize ‘reaction_time’ across coverages, routes, formations, dropbacks, and players. The main findings are: (1) zone coverages allow slightly higher expected catch rates than man, but shells and players vary widely in how early they trigger; (2) intermediate in‑breakers and deep shots draw much earlier triggers than quick game; and (3) certain linebackers, safeties, and nickel defenders repeatedly show up as very fast or very slow triggers in specific contexts, matching how they are used on the field.

**Introduction**

Passing in football is chaos in slow motion. The quarterback hits the top of his drop, the pocket shifts, receivers are breaking, and somewhere in the middle of that mess a defender decides, “It’s coming here.” Sometimes that decision happens after the ball leaves the hand. Just as often, it happens a beat earlier—on a route stem, a shoulder tilt, or a QB tell. This project is about that full window around the throw, before and after release, and what coverage defenders are doing as they trigger on the pass.
Good coverage players don’t just run fast, they react fast. They read the concept, trust what they see, and drive on the targeted receiver. That behavior shows up in the tracking data both before and after the ball is released. The goal is to turn that into something measurable: a ‘reaction_time’ metric that can take negative values (early triggers) as well as positive ones, and a catch‑probability model that shows how much that timing changes the odds of a completion.

Data and Setup

The analysis uses three components: pre‑throw tracking, post‑throw tracking, and a play‑level context table. For each play, the last pre‑throw frame is treated as the throw and ‘time_since_throw’ is defined in seconds relative to that moment: negative before release, positive after. Using the number of post‑throw frames, I compute ‘time_in_air’ and ‘time_to_arrival’ (time until the ball reaches its landing spot).
I isolate the targeted receiver, keep their position, timing, ball landing spot, and outcome, and join in play context (route, formation, coverage, etc.). For each targeted‑receiver frame, I pair in all defenders on the field, compute distances, and keep the nearest‑defender separation. This frame‑level table is the base for both the catch‑probability model and the ‘reaction_time’ metric.

**Modeling**

Baseline catch probability

The baseline model lives on the arrival frame (‘time_to_arrival’ = 0): one row per pass at the instant the ball gets there. Features are receiver location, ball landing location, distance receiver to ball, distance to nearest defender, and ball flight time.
The target variable is ‘is_complete’. After dropping plays with missing feature values, I split into train/validation/test, standardize numeric features, and fit logistic regression, a decision tree, a random forest, and XGBoost. XGBoost gives the best mix of AUC and calibration, so its predictions become the baseline ‘catch_probability’. Screens and flats show the highest expected catch probabilities, many intermediate routes cluster near the global mean, and deep corners, posts, gos, and wheels sit clearly lower. Across the board, zone coverages allow higher expected catch rates than man (outside of prevent coverage). 


Adding ‘reaction_time’

Next, the logistic model is expanded to include ‘reaction_time’ and simple receiver traits (height, weight, position), using the same split and preprocessing. Logistic regression is used because it models a binary outcome and produces interpretable completion probabilities from the input features. In an L1‑penalized logistic regression on standardized features, ‘reaction_time’ keeps a non‑zero coefficient. A one standard deviation decrease in ‘reaction_time’ reduces completion probability by about 15%.


**Defining Reaction in the Tracking**

A defender is said to have “reacted” when two things happen at the same time:

1. Defender–receiver distance begins a sustained decrease, defined as shrinking for 4 consecutive frames (0.4 seconds at 10 Hz). The first frame in that run is labeled as the moment of reaction.

2. The defender’s acceleration along the defender‑to‑receiver line on that frame is above the 70th percentile of all such accelerations in the dataset.

This definition is meant to capture true breaks on the route rather than noisy movement, so that a “reaction” reflects the defender driving toward where the route is actually going to finish, not a momentary close‑up before getting burned on a double move. Without the first criterion, there is a real chance the defender is accelerating in the wrong direction or only briefly closing before getting beat, which would look like a quick reaction but in practice increases completion probability. The second criterion makes sure the defender is actually speeding up toward the receiver, not just drifting in that direction. Because time is measured relative to the throw, a ‘reaction_time’ of −1.5 means the defender started closing 1.5 seconds before the ball left the quarterback’s hand. For each defender on each play, the earliest frame that meets both conditions is recorded as ‘reaction_time’.

**How Reaction Time Behaves Across Context**

Coverage

At the top level, average ‘reaction_time’ in man and zone is similar. Reactions by specific coverage type gleans more interesting insights. In Cover 0, defenders are in pure man with no post help, so corners and safeties are coached to play through the receiver and protect against immediate vertical wins rather than driving aggressively on the first hint of a break. That often delays the start of a sustained close and a true acceleration toward the receiver, which pushes ‘reaction_time’ closer to zero even though the coverage is technically the most aggressive on paper.​ In contrast, man coverages with help (like Cover 1 and 2‑man) let underneath defenders trigger earlier because they can play more aggressively on in‑breakers and intermediate routes, knowing there is a safety behind them. 

Routes

By route concept, ‘reaction_time’ mostly reflects how much time and freedom defenders have to trigger. Screens, flats, and many slants sit near zero or slightly positive because the ball is out almost as soon as the route declares; defenders may read it on time, but there is not enough runway to build the sustained close and acceleration this definition requires before the throw, so the first true “reaction” often starts with the ball already in the air.
Angles, crosses, and wheels live mostly in the negative range because they break into spaces owned by players who are allowed to be aggressive once they recognize the concept. Inside linebackers and safeties in hook and curl windows can read through the quarterback, see the angle or crosser bending into their zone, and drive before the throw without blowing their leverage. On wheel routes, covering linebackers who see the back release and turn up the sideline know there is no longer a run threat, which allows them freedom to trigger. 
Vertical routes like corners, posts, and gos land a bit negative for a different reason: deep‑responsibility corners and hybrid safeties must cap the vertical stem first, but once they trust the release, they flip and accelerate into the route. Reaction_time timestamps that full commitment toward where the route is going to finish.


Formation and Dropback

Next, I look at formation and quarterback movement to add an offensive layer. ‘Reaction_time’ is more negative (faster) in heavy sets, but the more interesting detail is who is actually triggering: in every formation except pistol, at least two of the five fastest reactors are linebackers, while in pistol that number drops to zero. 
Heavier, traditional sets like SINGLEBACK, I‑FORM, and JUMBO give linebackers a clear under‑center run picture and well‑defined rules, so they are free to be the first movers. When the back is deep and the quarterback is under center, the offense is mostly saying “run or classic boot,” which lets linebackers trigger downhill and then handle boot or play‑action as a secondary job. In that world, it makes sense that most of the fastest triggers in those formations are linebackers.
Pistol flips that picture. The back sits directly behind the quarterback, the formation is balanced, and the path still threatens true downhill run while the quarterback keeps shotgun‑style vision and option/RPO mechanics. That look hides run direction, keeps every gap live, and lets the offense pair those runs with quick play‑action or RPOs thrown into the same underneath windows that linebackers are coached to trigger toward. Box linebackers are asked to stay square, stay in their fits, and sort out run versus pass before firing, so the players who show up as the fastest reactors in pistol are the safeties and nickels on the edges of that conflict, not the linebackers stuck in the middle of it.
EMPTY and SHOTGUN pull ‘reaction_time’ closer to zero; with more vertical threats and space, defenders cannot drive on the first movement without risking explosives. Additionally, scramble and rollout plays show some of the earliest ‘reaction_time’ values. Corners and safeties who consistently lead those lists are the ones who plaster and drive as soon as the quarterback breaks the pocket (however, I would expect that the longer a play extends, the less important the initial reaction time becomes to predicting the probability of a completion). 




**Player Reaction Profiles**

Micah McFadden, Jack Gibbens, Kaden Elliss, Ja’Whaun Bentley, and Jack Campbell show up as the five fastest reactors in the sample, and all of them are off‑ball linebackers. For a coaching staff, this list is a trigger profile, not a final grade. It flags which linebackers consistently see it early and commit, so you can:
Lean into their strengths by calling more match‑heavy coverages, green‑dog rules, and aggressive underneath techniques for them in situations where early breaks actually matter.
Adjust usage by putting these players in hook, curl, and low‑hole help roles on key downs—especially against in‑breakers and heavier formations—rather than parking them in low‑leverage flat drops.
Pair their timing data with tape and production; if a fast‑trigger linebacker also tackles well and limits explosives, that supports an every‑down role, while frequent busts point to a coaching point of “aim the aggression” rather than “go faster.”
For a front office, the same list works as a scouting filter. Instead of only asking “who makes plays,” you can look for prospects and veterans whose timing profiles resemble these five. Teams who run schemes that lean on match principles should prioritize these players, because second‑level defenders who can handle conflict and trigger fast can help reduce expected completion percentage.


**Conclusion**

Reaction_time gives teams a way to scout, call, and deploy coverage with timing in mind. It helps identify which defenders trigger earliest, which structures and routes invite those early breaks, and how often that extra beat of anticipation turns catches into incompletions.

