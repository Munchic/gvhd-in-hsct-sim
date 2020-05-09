# Final project: network simulation

#### Simulation of graft-host-leukemia interactions after hematopoietic stem cell transplantation for acute myeloid leukemia

Source code: https://github.com/Munchic/cs166/blob/master/gvhd-gvl-simulation.ipynb

## I. Background

#### 1. About acute myeloid leukemia

Leukemia (blood cancer) develops from mutations that disrupt the normal development pathway of blood cells, often causing the overproduction of non-functional blood cells. American Cancer Society (2018) describes that acute myeloid leukemia (AML) is the most common type of leukemia and can occur in both children and adults. It is called myeloid leukemia because this disease affects the **myeloid progenitor cell** lineage (Fig. 1), which develops into erythrocytes (red blood cells) and essential cells of the innate immune system, responsible for immediate defense against pathogens. An example of innate immune cells affected under AML is neutrophils which make up about 60% of all white blood cells. In a bacterial infection, neutrophils are the first-responders that react to inflammatory signals and perform phagocytosis — engulfment and destruction of a pathogen. If the myeloid cells cannot mature (in AML), the patient can suffer from anemia (low red blood cells count and thus low oxygen delivery to organs) as well severely weakened immune system (e.g., from netropenia).

<img src="/Users/Munchic/Desktop/CS166/FP Fig 1.png" alt="As2 Fig 1" style="zoom:50%;" />

*Figure 1*. Hematopoietic stem cell lineage with focus on the myeloid progenitor cell specialization. Lymphoid progenitor cell lineage is hidden in this diagram. Original diagram by Rad and Häggström (2009).

Although it is a rare cancer (1% of all cancers), AML starts in the bone marrow, quickly moves to the blood, triggers one of the most aggressive cancers in terms of growth speed and resistance to apoptosis. The breadth of the white blood cells AML affects leads to the weakening of the patient’s innate immune system and prevents from ﬁghting oﬀ infections. Statistically, only 28.3% of patients survive after five years, according to the National Cancer Institute (2019).

#### 2. Modern treatment

A contemporary treatment for AML is chemotherapy followed by hematopoietic stem cell (HSC) transplantation. Chemotherapy is meant to wipe out old white blood cells (cancerous and healthy) to make place for HSCs transplantation from a donor. HSC transplant, often called a *graft* in a clinical setting, includes a fully-functional collection immune cells from a donor to help ﬁght oﬀ leukemia.

The most effective transplantation for AML is allogeneic (from other individuals) stem cell transplant from donors whose major histocompatibility complex (MHC) allele closely matches to the that of the receiver, such as from siblings or matched unrelated donor (Battipaglia et al., 2018). MHC (also called human leukocyte **antigen**) is a cell surface protein that present a peptide, a piece of protein manufactured within a cell, for effector immune cells to distinguish between infected (or non-self) and healthy cells. For example, killer CD8+ T cells can detect this antigen presentation and inject granzymes to explode to explode the infected cell (Fig. 2).

<img src="/Users/Munchic/Desktop/CS166/FP Fig 2.png" alt="img" style="zoom:50%;" />

*Figure 2*. Illustration CD8+ T-cell mediated cytotoxicity against an infected cell (cell with a foreign antigen). Original diagram by Dananguyen.

#### 3. Problem definition

A problem arises in selecting a donor due to variability in disease progression that depends on genetic compatibility of donor and recipient (host), speciﬁcally in terms of match of antigens presented on major histocompatibility complex (MHC) I and II. A mismatch in MHCs (usually in unrelated donor and host) means that graft-versus-leukemia (GvL) behavior is strengthened because the donor killer T cells would recognize the “foreign” cells and destroy them (Sweeney et al., 2019). On the other hand, a large mismatch in MHCs can lead to graft-versus-host disease (GvHD), a condition in which host cells are considered foreign and thus are destroyed (Fig. 2).

#### 4. Motivation for simulation

Since minimizing this overlap might be a great challenge of selecting the perfect donor, often methods like T cell depletion (reduction of donor T cells) helps with the situation. The modeling problem this ﬁnal project wants to address is that if we have donors of varying genetic similarity (in terms of MHC), how will levels of T cell depletion aﬀect the progression of GvL and GvHD.

#### 5. Glossary

- **GvHD** — graft-versus-host disease, tranplant attacking normal host cells
- **GvL** — graft-versus-leukemia, transplant attacking blood cancer cells
- **IS** — immune system
- [**H**]**SC**[**T**] — [hematopoietic] stem cell [transplant], healthy bone marrow transplant
- **MHC** — major histocompatibility complex, cell surface proteins to show cell identity
- **RBC** — red blood cell
- **AML** — acute myeloid leukemic
- **TCD** — T cell depletion

## II. Network architecture

#### 1. Overview and assumptions

This project aims to create a network simulation of a simpliﬁed immune system and human cells. Each node will represent a cell, and interaction could lead to cell destruction (node deletion/marking red) or cause more copies of the cell to reproduce ("self-interaction” of stem cells that can divide).

Cancer cells undergo fermentation in which they burn glucose rapidly (to convert to usable ATP energy) without using oxygen; thus, they can proliferate as long as there is energy source. Normal cells require oxygen for this same process, and they use up glucose at a lower pace; the oxygen is carried via red blood cells, a derivative of healthy myeloid cells.

When cancer myeloid cells progress, they will use up resources needed for healthy (donor) myeloid cells to develop. As a reminder, myeloid cells can develop to RBCs that carry oxygen to non-blood cells; therefore impeding development of normal myeloid cells will cause cell death in human host’s other non-blood cells. Additionally, myeloid cells can develop into parts of the innate immune system like neutrophils to protect the host from infections; again, impeding this development can cause cell death due to external pathogen.

In this simulation, there are various dynamics being modelled:

1. Interpopulation interactions upon introduction of different levels of graft CD8+ T cells
2. Intrapopulation interactions with cancer myeloid cells competing for resources (e.g., glucose) with healthy donor myeloid cells which carry oxygen to all healthy cells

There are several assumptions that distinguish this simplified immune system from the complex human immune system:

**Network representation assumptions:**

- **Similar cell size and mobility**: this simulation will assume that all cells will more or less interact with the Moore neighborhood in 3D (26 surrounding cells) with rewiring probability in a Watts-Strogatz network of 0.1 (to indicate some mobility);
- **Asynchronous edge-based update**: this simulation will interact cells with each other on one-on-one basis — only two cells can interact with each other at a time to simplify the update rule and computational load;
- **Constancy in network size**: this simulation will assume that the total number of cells with be the same throughout the simulation, and dead cells will be replenished by either type of cells (most likely AML if the cap is not reached yet) or remain a dead cell; interaction edges can rewire over time to simulate transport of blood particles in the host;
- **”Speaker"-first update**: since T cells are the main actors in the system (and they react to signalling to of other nodes), it makes sense to identify T cells, then their neighbors, and then perform interaction;

**Immune system assumptions**

- **Enclosed system**: only initialized (after chemotherapy and transplantation) immune and host cells can participate (destroy or multiply) in this system; there is no outflux or influx of cells;
- **Immune system abstraction**: because CD8+ T cells play the most important role in GvHD and GvL, we will represent as if they are the only component of the immune system and count other cells as “normal cells” without a particular functionality;
- **Linear genetic overlap**: MHC compatibility is qualified with a large number of genetic markers, each carrying a different contribution towards GvHD/GvL; to simplify, we will use percentages (e.g., $100\%$ = twin, $50\%$ = sibling or parent, $<0.01\%$ = completely unrelated donor);
- **No signalling pathways**: in a real IS, there are many cytokines (secreted substances that orchestrate immune cells), in this case we just assume that those pathways happen in the background, and relevant cells directly interact;
- **Self-replication only in multipotent stem cells**: this simulation will allow for indefinite self-replication of multipotent hematopoietic stem cells (like in real life) but no replication for myeloid progenitor cells (in real life, they have a limited number of replication cycles);
- **Homogeneity in a cell type population**: this simulation will assume that all cells of the same type will have the same quantitative abilities (e.g., need for oxygen/chance of destroying another cell). In reality, CD8+ T cells will have different T cell receptors (TCR) that determine their potency and affinity towards different kinds of antigens;
- **Constancy of HSCs in production and existence**: this simulation will assume that transplanted HSCs are just there (in the background) and don’t interact with any cell, they will just summon new donor CD8+ T cells and donor myeloid cells; in reality, HSCs divide indefinitely and some of them mature into specialized cells;
- **Simplistic T cell depletion**: this simulation will assume that T cell depletion just reduces the potency (cytotoxicity) of T cells, in reality in can be extraction of certain type of T cell depending on patient condition;

#### 2. Cellular agents

1. **Normal non-blood human cells**: these will be the majority of the system (including red blood count), they can get destroyed by donor T cells depending on conditions;
2. **Host cancerous myeloid cells**: chemotherapy will not wipe out old myeloid cells completely, so some will remain and try to proliferate; 
3. **Host normal T cells**: chemotherapy will not wipe out old white blood cells completely, so some will remain and try to proliferate (e.g., host T cells). These can help ﬁght oﬀ cancerous myeloid cells but they can also ﬁght foreign donor T cells;
4. **Donor T cells**: these will be the main area of interest, how much of them in the system will lead to GvL or GvHD, how much do we need given a genetic mismatch to stay in healthy equilibrium the longest;
5. **Donor myeloid cells**: these will be the main area of interest, how much of them in the system will lead to GvL or GvHD, how much do we need given a genetic mismatch to stay in healthy equilibrium the longest;

#### 3. Interactions

The interaction will be between two cells at a time and asynchronously updated — this will resemble the random (not really coordinated) interactions that happen within the blood stream. The type of interaction and outcomes will differ depending on what type of agents interact. Apart from interaction of cellular agents, there are global resources (Fig. 3) that the cells compete for to proliferate. 

<img src="/Users/Munchic/Desktop/CS166/Actual figure.png" alt="Screen Shot 2020-04-24 at 19.39.23" style="zoom:67%;" />

*Figure 3*. Interaction diagram of cellular agents in the network simulation. This figure abstracts different types of myeloid cells (e.g., red blood cell/neutrophil) to one type meant to support non-blood cells in a human host.

Let us describe in detail the types of interaction for agents of this system:

1. **Blood and blood**
   1. **Cancer myeloid cells and donor T cells**: upon interaction with donor T cells, depending on genetic mismatch, AML cells will be destroyed. Again, higher mismatch means higher chance of getting destroyed;
   2. **Cancer myeloid cells and host T cells**: upon interaction with host T cells, with a small chance (due to high MHC similarity), the cancer myeloid cells will be destroyed;
   3. **Donor T cells and host T cells**: upon interaction, depending on genetic mismatch, either one of the two can be destroyed or can be left untouched;
2. **Blood and non-blood**
   1. **Cancer myeloid cells and normal cells**: an interaction between these cells will be ignored since they are not directly affecting each other
   2. **Cancer myeloid cells and donor myeloid cells**: cancer cells are known to uptake resources and create acidity in the local environment which induces inflammation which induces cancer spread (metastasis); therefore, there will be a chance that donor myeloid can turn into a cancer cell (with higher chance where there is a larger saturation of cancer myeloid)

#### 4. Parameters

Genetic overlap of AML cells with donor T cells and healthy cells with donor T cells will be available to set as parameters to the model.

1. **Fixed conditioning variables for a baseline simulation**
   1. **Normal non-blood cells count** ($N_{norm} \in \mathbb{N}$ or `cap_cell_count`): specifies how many normal cells exist in the beginning (and the maximum capacity);
   2. **Post-chemo AML cells count** ($N_{AML} \in \mathbb{N}$ or `cell_aml_count`): specifies how many AML cells exist in the beginning;
   3. **Maximal myeloid capacity** ($N_{myeloid\_cap}$ or `cap_myeloid`): myeloid cells are produced from the hematopoietic stem cells located in the bone marrow; due to limit of resources (e.g., glucose), higher growth of myeloid leukemia cells will impede normal myeloid cell maturation;
   4. **Relative potency of AML cells** ($p_{rel\_potency} \in \mathbb{R}$ or `rel_aml_potency`): given unregulated growth and that the myeloid capacity is not reached, how more likely the next myeloid cell will be an AML rather than a healthy donor myeloid
   5. **Host cell death threshold** ($\mu_{cell\_death} < 1- \dfrac{N_{AML}}{N_{myeloid\_cap}}\in [0, 1] $ or `thres_cell_death`): threshold of percentage of heatlhy myeloid cells as a total of myeloid capacity, below which the innate immune system and oxygen will be compromised, leading to cell death;
   6. **Cell death chance** ($p_{cell\_death}$ or `p_cell_death`): probability of cell death in an asynchronous update when the host has reached the cell death threshold
   7. **T cell to cancer myeloid cell ratio** ($k_{transplant}=\dfrac{N_{Tcell}}{N_{AML}}$ or `ratio_t_myeloid`): ratio of counts of T cell in a **non-depleted** transplant to the number of myeloid cells present in the transplant;
8. **Baseline cytotoxicity of T cells** (`p_kill_baseline`): probability that CD8+ T cell will kill a foreign cell at 0% MHC overlap
   
2. **Independent “toggle” variables**
   1. **MHC compatibility** ($MHC_{overlap} \in [0, 1]$ or `mhc_overlap_perc`):
   2. **T cell depletion** ($N_{deplete} \in [0, 1]$ or `t_deplete_perc`):

#### 5. Metrics

1. **Overall survival** ($T_{survival}$ or `survival_overall`): assuming that a patient can only survive when there’s $30\%$ or more of normal cells, this will be the time when the normal cell count dips below $50\%$ of capacity $N_{norm}$; the higher, the better
2. **Survival without GvHD** ($T_{-GvHD}$ or `survival_no_gvhd`): GvHD will be counted as strong inflammatory response leading to less than $ 60\%$ of normal cells; the higher, the better
3. **Survival without relapse** ($T_{-GvL}$ or `survival_no_relapse`): relapse will be counted as AML cells overcompeting donor myeloid cells (when donor myeloid drop below $\mu_{cell\_death}$) and when AML takes over $60\%$ of the myeloid capacity $N_{myeloid\_cap}$; the higher, the better
4. **Remission time **($T_{remission}$ or `time_remission`): time until AML is mostly cleared ($0 \leq N_{AML} \leq 1$); the lower the better

## III. Evaluation scenarios

1. **No HSCT intervention**:

   Most likely the patient will die from relapse very quickly (due to lack of working RBCs).

2. **Non-depleted HSCT**:

   Depends on MHC compatibility, but generally there would be a lot of inflammation since donor T cells will count the host cells as foreign and attack it, we would see an early GvHD here. Should be paired with high MHC compatibility to reduce GvHD — in other words, if we have a well-matched donor (e.g., twin/sibling), we might not need to do TCD.

3. **High T cell after depletion in HSCT (low TCD)**:

   Similar to scenario 2, but a bit delayed GvHD. Still, should be paired with high MHC compatibility to reduce GvHD.

4. **Moderate T cell after depletion in HSCT (medium TCD)** :

   This one should be the best in terms of overall survival because there will be a delay in GvHD while maintaining GvL (delay in relapse). Should be paired with average MHC compatibility maintain GvL and prolong time to relapse.

5. **Low T cell after depletion in HSCT (high TCD)**:

   This one will be close to scenario 1 where relapse can happen early because donor T cells are not working as well. Should be paired with low MHC compatibility maintain GvL and prolong time to relapse. In other words, if you deplete too many T cells, they better be very violent.

## IV. Results

#### Cross-comparison of results

Two best results for each statistic is highlighted in green.

1. Overall survival:

   ![Screen Shot 2020-04-24 at 21.07.44](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 21.07.44.png)

   We can see here that the no HSCT seems to be doing well in terms of overall survival. While this makes sense in terms of not having GvHD, cancer will progress very fast in this case and will kill the patient quickly (like how it happens with AML). This is different from reality meaning that the parameters for the simulation are not entirely correct. On the other hand, it does show that some “middle” solution like low TCD (high T cell transplant) and high MHC compatibility works well. This is true in reality when people try to "match donor” — this means they are picking someone close in terms of MHC type.

2. Survival without GvHD:

   ![Screen Shot 2020-04-24 at 21.07.53](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 21.07.53.png)

   We see that when there is high MHC compatibility in no TCD or low TCD, it helps with GvHD. If we look at no TCD and low MHC compatibility, the result is much worse (in fact, even worse than high TCD, high MHC). This indicates that MHC compatibility plays an important role in preventing graft versus host attacks which can be detrimental to health. It is interesting to note that combinations like high TCD and low MHC compatibility don’t yield a good result; this might be because host T cells are too potent against donor's T cells and thus working counterproductively. In other words, there is a smaller total of T cells fighting against cancer (see suppl. fig. 5.1).

3. Survival without relapse:

   ![Screen Shot 2020-04-24 at 21.08.18](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 21.08.18.png)

   For relapse, we see a slightly opposing result to GvHD. This makes sense because if graft and host are MHC-similar, host’s cancer cells will also be less visible to the graft, as they are to the host cells. Therefore, increasing the MHC difference consistently allows for reduction of relapse (at the tradeoff of increased GvHD).

4. Remission time (faster better):

   ![Screen Shot 2020-04-24 at 21.08.24](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 21.08.24.png)

   There is almost no remission when we don’t do HSCT because the host cells are not great at recognizing cancer (due to very high MHC similarity and also biological reasons like cancer cells hiding their MHC). The best results for remission is similar to best results for survival without relapse. That is — more different MHC will kill cancer cells faster.

5. Conclusions:

   There is no obvious conclusion, however, we can observe a non-linear trade-off between T cell depletion and MHC compatibility. Generally, it seems like HSCT with low T cell depletion and high MHC compatibility seems to thrive the best. This is quite similar to the fact that in real life, there is matched donor seems to bring the most benefit to the patient (Battipaglia et al., 2018).

#### Example results

1. **No HSCT intervention**: See supplementary fig. 1 for an example run.
2. **Non-depleted HSCT**
   1. Low MHC compatibility: See supplementary fig. 2.1 for an example run.
   2. High MHC compatibility: See supplementary fig. 2.2 for an example run.

3. **Low T cell depletion in HSCT**
   1. Low MHC compatibility: See supplementary fig. 3.1 for an example run.
   2. High MHC compatibility: See supplementary fig. 3.2 for an example run.

4. **Moderate T cell depletion in HSCT**
   1. Low MHC compatibility: See supplementary fig. 4.1 for an example run.
   2. High MHC compatibility: See supplementary fig. 4.2 for an example run.

5. **High T cell depletion in HSCT**
   1. Low MHC compatibility: See supplementary fig. 5.1 for an example run.
   2. High MHC compatibility: See supplementary fig. 5.2 for an example run.

## V. Discussion

Network simulations of an immune system allow us to validate basic science principles like the relationship between MHC compatibility, T cell depletion, and health outcomes, such as GvHD or relapse. While it is computationally hard and expensive to emulate a real immune system with all the signalling and inflammatory pathways, even an abstract model with a valid parameter choice can showcase trends observed in a larger system. This model serves as a “simple” baseline to the immune system that can be extended further with more types of cells and more data-backed parameter tuning.

## VI. Appendix

#### References

American Cancer Society (2018). What Is Acute Myeloid Leukemia (AML)? Retrieved from https://www.cancer.org/cancer/acute-myeloid-leukemia/about/what-is-aml.html

Battipaglia, G., Ruggeri, A., Labopin, M., Volin, L., Blaise, D., Socié, G., … Nagler, A. (2018). Refined graft-versus-host disease/relapse-free survival in transplant from HLA-identical related or unrelated donors in acute myeloid leukemia. Bone Marrow Transplantation. doi:10.1038/s41409-018-0169-6

Cancer.net (2020). Leukemia - Acute Myeloid - AML: Statistics. Retrieved from https://www.cancer.net /cancer-types/leukemia-acute-myeloid-aml/statistics

National Cancer Institute (2019). Cancer Stat Facts: Leukemia - Acute Myeloid Leukemia (AML). Retrieved from https://seer.cancer.gov/statfacts/html/amyl.html

Sweeney, C., & Vyas, P. (2019). The Graft-Versus-Leukemia Eﬀect in AML. Frontiers in Oncology, 9.* doi:10.3389/fonc.2019.01217

Rad, A., & Häggström, M. (2009). Simplified hematopoiesis. Retrieved from https://commons.wikimedia.org/wiki/File:Hematopoiesis_simple.svg

#### Learning outcomes and habits of mind

- **#cs166-interpretresults**: provided hypothesis on how the system was going to behave and came up with metrics for evaluation and compared to hypothesized results
- **#modeling**: clearly stated assumptions and abstractions of the system, maintained the main dynamic of how a real immune system works
- **#descriptivestats**: utilized confidence intervals and mean to compare outcomes of experiments across different models

## VII. Supplementary material

#### Annotation

**Coloring**

- Blue is host cell
- Green is donor cell
- Red is AML

**Node shape**

- Square is normal cell
- Circle is myeloid cell
- Traingle is T cell

#### Figures

![Screen Shot 2020-04-24 at 19.43.04](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 19.43.04.png)

*Supplementary Figure 1*. Immune system network evolution without donor transplantation. Due to lack of RBC carrying oxygen, normal cells die out quickly.

![Screen Shot 2020-04-24 at 19.59.01](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 19.59.01.png)

*Supplementary Figure 2.1*. Immune system network evolution with donor transplantation and no depletion. A very strong GvHD effect can be observed.

![Screen Shot 2020-04-24 at 20.01.29](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 20.01.29.png)

*Supplementary Figure 2.2*. Immune system network evolution with donor transplantation and no depletion. Higher MHC compatibility reduces the GvHD effect on normal cells compared to 2.1.

![Screen Shot 2020-04-24 at 20.46.08](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 20.46.08.png)

*Supplementary Figure 3.1*. Immune system network evolution with donor transplantation and low depletion. A very strong GvHD effect can be observed like in 2.1.

![Screen Shot 2020-04-24 at 20.47.04](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 20.47.04.png)

*Supplementary Figure 3.2*. Immune system network evolution with donor transplantation and low depletion. A still strong GvHD effect like in 3.1, although the GvHD progresses slower that in 3.1.

![Screen Shot 2020-04-24 at 20.49.39](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 20.49.39.png)

*Supplementary Figure 4.1*. Immune system network evolution with donor transplantation and medium depletion. Strong GvHD due to MHC-mismatch effect like in 3.1, but GvHD progresses slower that in 3.1.

![Screen Shot 2020-04-24 at 20.52.13](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 20.52.13.png)

*Supplementary Figure 4.2*. Immune system network evolution with donor transplantation and medium depletion. Should be lower GvHD than in 4.1, but could be by chance that looks more GvHD.

![Screen Shot 2020-04-24 at 20.55.01](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 20.55.01.png)

*Supplementary Figure 5.1*. Immune system network evolution with donor transplantation and high depletion. GvL is very weak here, and relapse happens (similar to figure 1).

![Screen Shot 2020-04-24 at 20.58.39](/Users/Munchic/Library/Application Support/typora-user-images/Screen Shot 2020-04-24 at 20.58.39.png)

*Supplementary Figure 5.2*. Immune system network evolution with donor transplantation and high depletion. GvL is very weak here, and relapse happens (similar to figure 1).