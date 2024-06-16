- [[Geometric ]]
- [[Evaluating Mechanisms]]
- ## Detailed Distribution Breakdown
	- ### Context and Setup
	  
	  We have:
	  1. A total token supply to be distributed across 5 tiers over 5 days.
	  2. The number of individuals in each tier.
	  3. The total tokens per tier.
	  4. A decay factor determining the reduction in tokens available each day.
	- ### Objective
	  
	  To distribute tokens such that each day, the number of tokens an individual can claim decreases, following an exponential decay model.
	- ### Formula for Daily Token Allocation
	  
	  The daily token allocation for an individual follows an exponential decay. The formula for the tokens available to claim by an individual on a given day is:
	  
	  $$
	  a_n = a_1 \cdot r^{(n-1)}
	  $$
	  
	  Where:
	- \( a_n \) is the number of tokens available to claim on day \( n \).
	- \( a_1 \) is the initial number of tokens available to claim on day 1.
	- \( r \) is the decay factor (a constant less than 1, representing the rate at which tokens decrease each day).
	- \( n \) is the day number (from 1 to 5).
	- ### Explanation of Each Formula Component
	- #### 1. Initial Tokens Available ( \( a_1 \) )
	  The initial number of tokens available to claim on day 1 for each tier is computed based on the total token supply for that tier, the number of individuals in that tier, and the decay over 5 days.
	  
	  We use the sum of a geometric series to solve for \( a_1 \):
	  
	  The sum of tokens an individual can claim over 5 days:
	  $$
	  S_5 = a_1 \cdot \frac{1 - r^5}{1 - r}
	  $$
	  
	  Where:
	- \( S_5 \) is the total number of tokens an individual can claim over 5 days.
	- \( \frac{1 - r^5}{1 - r} \) is the sum of the first 5 terms of a geometric sequence with the first term \( a_1 \) and common ratio \( r \).
	  
	  Given the total tokens for a tier and the number of individuals in that tier, the formula to find \( a_1 \) is:
	  $$
	  a_1 = \frac{\text{Total Tokens per Tier}}{\text{Number of Individuals} \cdot \frac{1 - r^5}{1 - r}}
	  $$
	- #### 2. Tokens Decrease Each Day ( \( r \) )
	  The decay factor \( r \) should be chosen to ensure a smooth exponential decay. In these examples, we used \( r = 0.8 \):
	- On Day 1, the tokens are \( a_1 \).
	- On Day 2, the tokens are \( a_1 \cdot 0.8 \).
	- On Day 3, the tokens are \( a_1 \cdot (0.8)^2 \).
	- And so on.
	- #### Example Calculation — Step-by-Step
	  
	  Let's take Tier 1 with 1,000 individuals and a total of 5,000,000 tokens as an example.
	- ##### Step 1: Calculate \( S_5 \)
	  
	  Sum of tokens an individual can claim over 5 days:
	  $$
	  S_5 = a_1 \cdot \frac{1 - (0.8)^5}{1 - 0.8}
	  $$
	  $$
	  S_5 = a_1 \cdot \frac{1 - 0.32768}{0.2}
	  $$
	  $$
	  S_5 = a_1 \cdot \frac{0.67232}{0.2}
	  $$
	  $$
	  S_5 = a_1 \cdot 3.3616
	  $$
	- ##### Step 2: Solve for \( a_1 \)
	  
	  Total tokens per tier = 5,000,000
	  Number of individuals in tier = 1,000
	  
	  We set up the equation:
	  $$
	  5,000,000 = 1,000 \cdot a_1 \cdot 3.3616
	  $$
	  $$
	  5,000,000 = a_1 \cdot 3,361.6
	  $$
	  $$
	  a_1 = \frac{5,000,000}{3,361.6} \approx 1,487.90
	  $$
	- ##### Step 3: Calculate Tokens Available Each Day
	- **Day 1**: \( a_1 = 1,487.90 \)
	- **Day 2**: \( 1,487.90 \cdot 0.8 \approx 1,190.32 \)
	- **Day 3**: \( 1,190.32 \cdot 0.8 \approx 952.26 \)
	- **Day 4**: \( 952.26 \cdot 0.8 \approx 761.81 \)
	- **Day 5**: \( 761.81 \cdot 0.8 \approx 609.45 \)
	  
	  This process is repeated for each tier with the appropriate number of individuals and total tokens allocated to each tier.
	- ### Summary
	  
	  The formula and process for generating the daily tokens available for claim, considering the total supply, decay factor, and number of individuals, involve:
	  
	  1. Determining the initial tokens available to claim on the first day by solving for \( a_1 \).
	  2. Applying the exponential decay formula \( a_n = a_1 \cdot r^{(n-1)} \) to get the tokens for subsequent days.
	  3. Summing the results across all days to ensure the total tokens match the predefined allocation for each tier.
	  
	  By following this method, the tokens decrease day by day, starting from the highest on the first day, ensuring a smooth and fair distribution over the period.
- ## Reasoning for 0.8 Decay Factor
	- The choice of decay factor \( r \) is critical to meeting specific distribution goals. The decay factor controls how the distribution of tokens decreases each day. Let's delve into the reasoning for using a decay factor of \( 0.8 \), explore other potential values, and outline criteria for choosing the best decay factor.
	- ### Reasoning Behind Using a Decay Factor of 0.8
	  
	  1. **Moderate Decrease Rate**:
		- A decay factor of \( 0.8 \) represents a moderate rate of decay, meaning that each subsequent day's allocation is reduced to 80% of the previous day's allocation.
		- This rate achieves a balance between distributing a significant number of tokens initially while ensuring a substantial reduction over the distribution period (5 days).
		  
		  2. **Manageable Token Allocation**:
		- With \( r = 0.8 \), each day's reduction in tokens does not create too steep a decline, ensuring that recipients receive a fair share over multiple days.
		- A balance in token distribution can help maintain the engagement of participants across all days.
		  
		  3. **Smooth Geometric Progression**:
		- A decay factor of \( 0.8 \) establishes a smooth geometric progression that is easy to compute and apply.
	- ### Exploring Other Values
	  
	  Other decay factors can be used to achieve different distribution behaviors. Let's explore a few alternatives:
	  
	  1. **Higher Decay Factor (Closer to 1)**:
		- Example: \( r = 0.9 \)
		- **Effect**: Slower decay with a more gradual reduction in daily tokens.
		- **Use Case**: When the goal is to maintain higher token allocations across the days, providing more balanced daily distributions.
		  
		  2. **Lower Decay Factor (Closer to 0)**:
		- Example: \( r = 0.5 \)
		- **Effect**: Faster decay with a significant reduction in daily tokens.
		- **Use Case**: When the goal is to reward early participants more heavily. Day 1 participants receive significantly more tokens than later days.
	- ### Criteria for Determining the Best Decay Factor
	  
	  1. **Distribution Goals**:
		- **Front-Loaded vs. Even Distribution**: Decide whether the goal is to distribute more tokens early (front-loaded) or to have a more even distribution across the days.
		- **Reward Consistency**: Consider if the participants should receive consistent rewards over the days or if there's a need to incentivize early claims.
		  
		  2. **Audience Engagement**:
		- **Participant Behavior**: Understand how the decay factor impacts participant engagement. A steep decay may lead to rush participation on the first day, whereas a gentler decay may sustain engagement across all days.
		  
		  3. **Total Tokens and Number of Days**:
		- Adjust the decay factor based on the total token supply and the number of distribution days. Longer distribution periods may benefit from smaller decay factors to avoid running out of tokens too quickly.
		  
		  4. **Mathematical Constraints**:
		- Ensure that the chosen decay factor causes the total token distribution to match the available supply over the specified days.
	- ### Calculating Other Decay Factors
	- #### Example: Decay Factor \( r = 0.9 \)
	- For \( r = 0.9 \):
		- The tokens for each day would reduce more slowly than with \( r = 0.8 \).
		  
		  Using the previous example (Tier 1, 1,000 individuals, 5,000,000 tokens):
		  
		  1. Calculate \( a_1 \) with \( r = 0.9 \):
		  
		  For \( r = 0.9 \):
		  $$
		  S_5 = a_1 \cdot \frac{1 - 0.9^5}{1 - 0.9} = a_1 \cdot 4.0951
		  $$
		  $$
		  5,000,000 = 1,000 \cdot a_1 \cdot 4.0951
		  $$
		  $$
		  a_1 = \frac{5,000,000}{1,000 \cdot 4.0951} \approx 1,220.17
		  $$
		  
		  2. Tokens Each Day (approximate values with \( a_1 = 1,220.17 \)):
	- Day 1: \( 1,220.17 \)
	- Day 2: \( 1,220.17 \cdot 0.9 \approx 1,098.15 \)
	- Day 3: \( 1,098.15 \cdot 0.9 \approx 988.3335 \)
	- Day 4: \( 988.3335 \cdot 0.9 \approx 889.50 \)
	- Day 5: \( 889.50 \cdot 0.9 \approx 800.55 \)
	- ### Conclusion
	  
	  Using a decay factor of \( 0.8 \) provides a balanced and moderate rate of decay suitable for many scenarios. However, different factors like participant behavior, engagement goals, and the desired rate of token distribution can suggest the use of different decay factors. A higher \( r \) may maintain token attractiveness over the distribution period, while a lower \( r \) better incentivizes early participation. The best choice depends on the specific objectives and constraints of the token distribution plan.
-