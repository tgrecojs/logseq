### Conditions
	- 1. **Epoch Duration**: 24 hours (instead of 36).
	- 2. **Token Decay**: As continuous decay over 24-hour epochs.
	- 3. **Tier-based distribution**: Different allocations for various tiers.
	- 4. **Expansion Mechanism**: Additional 1% of the token supply is added for every 32,500 claims, with 5% of this newly minted supply distributed to claimants.
- ## Continuous Decay for 24-hour Epochs
	- Using the continuous exponential decay formula:
	- $$ N(t) = N_0 \cdot e^{-\lambda t} $$
	- where:
	- - \( N(t) \) is the amount at time \( t \).
	- - \( N_0 \) is the initial amount.
	- - \( \lambda \) is the decay constant (rate of decay).
	- - \( t \) is the elapsed time.
	- To match a half-life of 24 hours, the decay constant \( \lambda \) is:
	- $$ \lambda = \frac{\ln(2)}{24} $$
-
- ## Token Allocation per Participant in Each Tier
	- 1. **Total Initial Supply**: \( X \).
	- 2. **Allocation Percentages for Each Tier**:
		- Tier 1: \( T_1 = 10\% \)
		- Tier 2: \( T_2 = 20\% \)
		- Tier 3: \( T_3 = 30\% \)
		- Tier 4: \( T_4 = 20\% \)
		- Tier 5: \( T_5 = 20\% \)
-
-
- ## Initial Allocation per Participant:
	- $$ A_i = \frac{X \cdot T_i}{N_i} $$
- ### Continuous Decay Over Time:
	- Allocation at any time \( t \) (in hours) within the epoch:
	- $$ D_{i}(t) = A_i \cdot e^{-\frac{\ln(2)}{24} t} $$
- ### Expansion Mechanism:
	- After every 32,500 claims within a given epoch, an additional 1% of the token supply (\( X \)) is added.
	- 5% of this newly minted 1% is distributed pro-rata to individuals who have already claimed.
-
- ### Updated Table and General Formula
	- For an initial total supply \( X \):
	- | Tier | Requirement Met | Initial Participants | Initial Allocation per IST | Formula at Time \( t \) (in hours) |
	  |-------|-------------------|----------------------|----------------------------------------|--------------------------------------------|
	  | Tier 1| 4/4 + General | 1,000 | \( A_1 = \frac{X \cdot 0.10}{1,000} \) | \( D_{1}(t) = A_1 \cdot e^{-\frac{\ln(2)}{24} t} \) |
	  | Tier 2| 3/4 + General | 5,000 | \( A_2 = \frac{X \cdot 0.20}{5,000} \) | \( D_{2}(t) = A_2 \cdot e^{-\frac{\ln(2)}{24} t} \) |
	  | Tier 3| 2/4 + General | 10,000 | \( A_3 = \frac{X \cdot 0.30}{10,000} \) | \( D_{3}(t) = A_3 \cdot e^{-\frac{\ln(2)}{24} t} \) |
	  | Tier 4| 1/4 + General | 40,000 | \( A_4 = \frac{X \cdot 0.20}{40,000} \) | \( D_{4}(t) = A_4 \cdot e^{-\frac{\ln(2)}{24} t} \) |
	  | Tier 5| General only | 100,000 | \( A_5 = \frac{X \cdot 0.20}{100,000} \) | \( D_{5}(t) = A_5 \cdot e^{-\frac{\ln(2)}{24} t} \) |
- ### Example Applied to 10 Trillion Tribbles (\( X = 10^{13} \))
	- **Initial Allocations**:
	- Tier 1: \( A_1 = \frac{10^{13} \cdot 0.10}{1,000} = 10^{12} \text{ Tribbles} \)
	- Tier 2: \( A_2 = \frac{10^{13} \cdot 0.20}{5,000} = 4 \cdot 10^{11} \text{ Tribbles} \)
	- Tier 3: \( A_3 = \frac{10^{13} \cdot 0.30}{10,000} = 3 \cdot 10^{11} \text{ Tribbles} \)
	- Tier 4: \( A_4 = \frac{10^{13} \cdot 0.20}{40,000} = 5 \cdot 10^{10} \text{ Tribbles} \)
	- Tier 5: \( A_5 = \frac{10^{13} \cdot 0.20}{100,000} = 2 \cdot 10^{10} \text{ Tribbles} \)
	- 2. **Allocation at Time \( t = 12 \) hours (mid-epoch)**:
	- For Tier 1:
	- $$ D_{1}(12) = 10^{12} \cdot e^{-\frac{\ln(2)}{24} \cdot 12} $$
	- $$ D_{1}(12) = 10^{12} \cdot e^{-\ln(2)/2} $$
	- $$ D_{1}(12) = 10^{12} \cdot \frac{1}{\sqrt{2}} $$
	- $$ D_{1}(12) = 10^{12} \cdot 0.707 \approx 7.07 \times 10^{11} \text{ Tribbles} $$
- ### Expansion Mechanism Implementation:
	- #### For every 32,500 claims, an additional 1% is minted:
	- $$ \text{Newly Minted Tokens} = 0.01 \times X $$
		- > 2. 5% of the newly minted tokens are distributed to existing claimants:
	- #### Total newly minted tokens per 32,500 claims:
	- $$ 0.01 \times X $$
	- #### Distribution to claimants:
	- $$ 0.05 \times (0.01 \times X) = 0.0005 \times X $$
	- > Per participant distribution depends on the number of claimants at the time of expansion.
-
-
- ## Overall Generalized Formula and Table
	- ### Formula:
		- #### **Initial Allocation**:
	- $$ A_i = \frac{X \cdot T_i}{N_i} $$
	- #### **Continuous Decay**:
	- $$ D_{i}(t) = A_i \cdot e^{-\frac{\ln(2)}{24} t} $$
	- #### **Additional Distribution Per Claimant**:
		- Every 32,500 claims:
		- $$ \frac{0.0005 \times X}{\text{Number of Claimants}} $$
	- ### continuous decay and additional distribution table:
		- > This model provides a detailed structure for continuous decay of token allocation over a 24-hour epoch and includes the mechanism for supply expansion based on claim activity.
		- | Tier | Requirement Met | Initial Participants | Initial Allocation per IST | Formula at Time \( t \) (in hours) | Additional Distribution per 32,500 Claims |
		  |-------|-------------------|----------------------|----------------------------------------|--------------------------------------------|--------------------------------------------------|
		  | Tier 1| 4/4 + General | 1,000 | \( A_1 = \frac{X \cdot 0.10}{1,000} \) | \( D_{1}(t) = A_1 \cdot e^{-\frac{\ln(2)}{24} t} \) | \( 0.0005 \times X / \text{Claimants} \) |
		  | Tier 2| 3/4 + General | 5,000 | \( A_2 = \frac{X \cdot 0.20}{5,000} \) | \( D_{2}(t) = A_2 \cdot e^{-\frac{\ln(2)}{24} t} \) | \( 0.0005 \times X / \text{Claimants} \) |
		  | Tier 3| 2/4 + General | 10,000 | \( A_3 = \frac{X \cdot 0.30}{10,000} \) | \( D_{3}(t) = A_3 \cdot e^{-\frac{\ln(2)}{24} t} \) | \( 0.0005 \times X / \text{Claimants} \) |
		  | Tier 4| 1/4 + General | 40,000 | \( A_4 = \frac{X \cdot 0.20}{40,000} \) | \( D_{4}(t) = A_4 \cdot e^{-\frac{\ln(2)}{24} t} \) | \( 0.0005 \times X / \text{Claimants} \) |
		  | Tier 5| General only | 100,000 | \( A_5 = \frac{X \cdot 0.20}{100,000} \) | \( D_{5}(t) = A_5 \cdot e^{-\frac{\ln(2)}{24} t} \) | \( 0.0005 \times X / \text{Claimants} \) |
	-