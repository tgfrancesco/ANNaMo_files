# Forward Flux Sampling with ANNaMo

This folder contains all the necessary input files to evaluate the rate of a toehold-mediated strand displacement process of a system composed of a 20-nucleotide-long incumbent strand that is complementary to a substrate strand that has a toehold of 15 nucleotides and that can be displaced by a 35-nucleotide-long invader strand. Modelled with ANNaMo, the lengths (in beads) of the incumbent, substrand and invader strands are 7, 12 and 12, respectively.

## Background

Here we sample the rate at which the systems goes from state $A$ (the incumbent is bonded to the substrate) to state $B$ (the incumbent has been completely displaced by the invader). We first of all define a set of interfaces $\lbrace \lambda_{Q_i}^{{Q_i}+1}\rbrace$ through which a trajectory has to pass to go from $A$ to $B$ by defining an order parameter $Q$ whose value depends on the relative distances between the beads, and on the bonding pattern (*i.e.* who's bonded with whom). In particular, defining $r_{\rm min}$ as the minimum distance between pairs of beads that are bonded with each other in either $A$ or $B$ and $n_b$ is the number of bonds, the value of the order parameter is

* $Q = -2$ if $r_{\rm min} > 4\, \sigma$, where $\sigma$ is the diameter of a bead;
* $Q = -1$ if $4 \, \sigma \geq r_{\rm min} > 1.6\, \sigma$;
* $Q = 0$ if $r_{\rm min} < 1.6\, \sigma$ and $n_b = 0$;
* $Q = n_B$ otherwise;

If $\Phi(A)$ is the flux across $\lambda_{-1}^0$ starting from a state $Q = -2$, the overall $A \to B$ rate is

$$
R = \Phi(A) \prod_{Q=1}^{Q=Q_{\rm max}} P(\lambda_{Q-1}^Q|\lambda_{Q-2}^{Q-1})
$$

where $P(\lambda_{Q-1}^Q|\lambda_{Q-2}^{Q-1})$ is the probability that a trajectory that has crossed the $\lambda_{Q-2}^{Q-1}$ interface crosses the $\lambda_{Q-1}^Q$ interface before returning back to $A$.

In practice, $\Phi(A)$ is evaluated by starting $N$ configurations in $A$ and computing the rate of crossing $\lambda_{-1}^0$, while $P(\lambda_{Q-1}^Q|\lambda_{Q-2}^{Q-1})$ is estimated by launching $N$ simulations from state $Q-1$ and counting how many reach state $Q$.

## Implementation

The calculation is carried out with two scripts, `ffs_flux.py`, which estimates $\Phi(A)$, and `ffs_shoot.py` that can be used to obtain the conditional probability $P(\lambda_{Q-1}^Q|\lambda_{Q-2}^{Q-1})$. Both scripts take as input a [TOML](https://toml.io/en/) file that specifies how it should run. For the specific system at hand we define 5 interfaces, one for each folder:

* In `FLUX` we evaluate $\Phi(A)$ through the interface $(-1, 0)$
* In `SHOOT_1` we evaluate the conditional probability of crossing $(0, 1)$
* In `SHOOT_2` we evaluate the conditional probability of crossing $(4, 5)$, when the last bead of the toehold binds to the invader
* In `SHOOT_3` we evaluate the conditional probability of crossing $(5, 6)$, when the first bead of the incumbent is displaced
* In `SHOOT_4` we evaluate the conditional probability of crossing $(11, 12)$​, when the incumbent is fully displaced

Note that the conditional probability of the two latter crossings are $\approx 1$ (*i.e* the process is spontaneous) and therefore it is not strictly necessary to evaluate them. We left them in the example for completeness.

The whole example can be run by executing `bash run.sh`. Note that this would take many days on a 16-core machine, so it's better to run each simulation by hand.

**Nota Bene:** simulations must be run sequentially, since each but the first one (`FLUX`) uses configurations generated by the previous one.

The `analyse.sh` script can be used to obtain an estimate of the rate. The output is printed on screen in terms of rate $R$ and steps (defined as $1/R$). The same values (accompanied by its associated error) is also printed on the `rate.dat` and `steps.dat` files.

The options in the `ffs.toml` files should be self-explanatory, but here is a short list of the main ones:

* `input` path to the oxDNA input file
* `initial_seed` is the seed used to generate the seeds for individual simulations
* `ncpus` number of processes (*i.e.* concurrent simulations)
* `interface` defines the target interface
* `desired_success_count` (only for `ffs_flux.py`) number of starting configurations to be generated
* `total_simulations` (only for `ffs_shoot.py` number of simulations that will be run starting from randomly-chosen configurations generated during the crossing of the previous interface
* `starting_conf_pattern` the path where the previous step's configuration files reside
