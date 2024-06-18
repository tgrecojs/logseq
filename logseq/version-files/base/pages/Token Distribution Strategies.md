## Epoch Decay Mechanism
- #### Desired Properties
	- 1. **Distribution Duration**:
		- The strategy will take place over five days.
	- 2. **Even Distribution with Incentives**:
		- We aim for a distribution mechanism that avoids a sharp decrease in supply either at the campaign's start or end.
		- We also incorporate incentives to encourage early claims.
	- 3. **Token Supply**:
		- The total token supply will range from 10 to 20 million to ensure scarcity.
	- 4. **Incentivizing Early Claims**:
		- To incentivize early claims, we will include a decay rate that modestly decreases daily token allocations.
		- Additionally, a "bonus mint" event is planned to further encourage prompt participation.
- #### Decay Factor Considerations
	- Using a decay factor of 0.85% aligns somewhat well with our focus on having a decay slope that is steep enough to motivate users to take action, but is not steep enough to end up in a scenario where a disporportionate number of tokens are allocated to a small, select number of people.
	- My intention is to strike a balance between adequately incentivizing first-movers  somewhat even distribution
- ### Distribution Mechanics
	- #### Step 1: Initial Token Allocation
		- We'll allocate the total token supply equally among the 5 tiers.
		- **Example Token Supply**: 15,000,000
		- **Tokens per Tier**: \( \text{Total Tokens} / 5 = 15,000,000 / 5 = 3,000,000 \)
	- #### Step 2: Calculation of Daily Tokens
		- Using the decay formula:
			- $$
			  a_n = a_1 \cdot 0.85^{(n-1)}
			  $$
		- Where \( a_n \) is the number of tokens available on day \( n \). The total allocation for each individual is:
			- $$
			  S_5 = a_1 \cdot \frac{1 - 0.85^5}{1 - 0.85}
			  $$
		- For 3,000,000 tokens distributed to 1,000 individuals in Tier 1:
			- $$
			  3,000,000 = 1,000 \cdot a_1 \cdot 3.953
			  $$
			  $$
			  a_1 = \frac{3,000,000}{1,000 \cdot 3.953} \approx 759.19
			  $$
	- #### Step 3: Tokens Claimable Each Day
		- For each tier, tokens claimable per individual:
		- | Day | Tokens (Tier 1)  | Tokens (Tier 2)  | Tokens (Tier 3)  | Tokens (Tier 4)   | Tokens (Tier 5)  |
		  |-----|------------------|------------------|------------------|------------------|------------------|
		  | 1   | 759.19           | 151.84           | 75.92            | 18.98            | 7.59             |
		  | 2   | 759.19 × 0.85 ≈ 645.32 | 151.84 × 0.85 ≈ 129.06 | 75.92 × 0.85 ≈ 64.53 | 18.98 × 0.85 ≈ 16.13 | 7.59 × 0.85 ≈ 6.45  |
		  | 3   | 645.32 × 0.85 ≈ 548.52 | 129.06 × 0.85 ≈ 109.71 | 64.53 × 0.85 ≈ 54.85 | 16.13 × 0.85 ≈ 13.71 | 6.45 × 0.85 ≈ 5.48  |
		  | 4   | 548.52 × 0.85 ≈ 466.24 | 109.71 × 0.85 ≈ 93.25  | 54.85 × 0.85 ≈ 46.62  | 13.71 × 0.85 ≈ 11.65 | 5.48 × 0.85 ≈ 4.66   |
		  | 5   | 466.24 × 0.85 ≈ 396.30  | 93.25 × 0.85 ≈ 79.26 | 46.62 × 0.85 ≈ 39.63  | 11.65 × 0.85 ≈ 9.90 | 4.66 × 0.85 ≈ 3.96    |
-
- ### Bonus Mint Event
	- 1. 5% of total tokens set aside for bonuses.
	- 2. **Trigger Conditions**:
		- If a certain threshold of claims is met within an epoch, initiate a bonus mint.
		- An epoch could be defined as a week or another time frame.
	- 3. **Reward Distribution**:
		- Bonus adds 1% of the token supply.
		- Half of this 1% is distributed to users who met the claim target retroactively.
	-
- ## Incremental Decay Mechanism
- ### Concept
	- The incremental decay mechanism is designed to further incentivize early claims by progressively reducing the number of tokens available for each subsequent claimant within a tier. Under this mechanism, the first claimant within a tier receives the maximum token amount for that epoch, and each subsequent claimant receives slightly fewer tokens than the previous one.
- ### Key Elements
	- 1. **Initial Distribution**:
		- The number of tokens available for the first claimant in each epoch.
	- 2. **Decay Factor**:
		- The rate at which tokens decrease for each subsequent claimant within the same epoch.
	- 3. **Resulting Token Allocations**:
		- The actual number of tokens each user receives based on their claim order.
- ### Mathematics of Incremental Decay
	- 1. **Initial Token Amount**:
		- We'll denote \(a_1\) as the number of tokens the first claimant receives. For this example, \(a_1 = 1,000\) tokens for the first day for Tier 1.
	- 2. **Incremental Decay Factor** (\(r_i\)):
		- The decay factor \(r_i\) determines the reduction for each subsequent claim. For this example, we use \(r_i = 0.99\).
- ### Formula for Token Allocation
	- The token amount \(a_k\) available to the \(k\)-th claimant is calculated using the formula:
	- $$
	  a_k = a_1 \cdot r_i^{(k-1)}
	  $$
	- **Where**:
		- \(a_k\) = Tokens for the \(k\)-th claimant.
		- \(a_1\) = Initial tokens for the first claimant (1,000 tokens for Tier 1).
		- \(r_i\) = Incremental decay factor (0.99).
		- \(k\) = Claimant position (1, 2, 3, ...).
- ### Detailed Calculation for Incremental Decay
- #### Example: Day 1, Tier 1
	- 1. **First Claimant**:
		- \(a_1 = 1,000\)
	- 2. **Second Claimant**:
		- \(a_2 = a_1 \cdot r_i^{(2-1)} = 1,000 \cdot 0.99 = 990\)
	- 3. **Third Claimant**:
		- \(a_3 = a_2 \cdot r_i^{(3-1)} = 990 \cdot 0.99 = 980.1\)
	- This pattern continues for all subsequent claimants.
		- | Claimant | Tokens Distributed (Day 1, Tier 1) |
		  |----------|-------------------------------------|
		  | 1        | 1,000 (initial)                     |
		  | 2        | 1,000 × 0.99 = 990                  |
		  | 3        | 990 × 0.99 = 980.10                 |
		  | 4        | 980.10 × 0.99 = 970.30              |
		  | 5        | 970.30 × 0.99 = 960.59              |
		  | ...      | ...                                 |
		  | 1,000    | Calculated to fit within the total allocation |
- ### Summation and Fit within the Allocation
	- To ensure the method fits within the daily token allocation for the tier, we can use the formula for the sum of a geometric series:
		- $$
		  S_n = a_1 \cdot \frac{1 - r_i^n}{1 - r_i}
		  $$
		- **Where**:
- \(S_n\) = Total sum of tokens distributed to \(n\) claimants
- \(a_1 = 1,000\)
- \(r_i = 0.99\)
- \(n = 1,000\) claimants
- Calculating the total tokens allocated on Day 1:
- $$
  S_{1000} = 1,000 \cdot \frac{1 - (0.99)^{1000}}{1 - 0.99}
  $$
- Given \( (0.99)^{1000} \approx e^{-10} \approx 4.54 \times 10^{-5} \):
- $$
  S_{1000} = 1,000 \cdot \frac{1 - 4.54 \times 10^{-5}}{0.01}
  $$
- $$
  S_{1000} = 1,000 \cdot 99.99546 \approx 99,995.46
  $$
- Since the initial approximation values (eg: daily 1,000 tokens fitting higher distribution, slight variance balance) supplement calculations, rapid final sun guarantees and practical implementation validate overall projections.
- ### Chart for Incremental Decay
	- **Figure: Incremental Decay (First 100 Claims)**
		- ```plaintext
		  Claimant      Tokens
		  ---------     -------
		  1             1,000
		  2             990
		  3             980.1
		  4             970.3
		  ...           ...
		  100           ≈ 367.9
		  ```
- Initial high distributions drop-off per preceding users progressively maintain engaging, controlled, weight.
- ### Summary of the Incremental Decay Mechanism
	- **Incentivizes Early Claims**: Users acting soon receive favorable larger rewards.
	- **Fairly Balanced**: Decay factor gradual ensuring motivating, equitable share for successive users.
	- **Mathematical Integrity**: Total token calculations ensuring operationally validity, bounded maximal aligned distribution projection.
-