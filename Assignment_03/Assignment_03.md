Assignment 03 - Web Scraping and Text Mining
================
Qiushi Peng
2022-11-05

``` r
knitr::opts_chunk$set(echo = TRUE)
library(httr)
library(xml2)
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.6     ✔ purrr   0.3.4
    ## ✔ tibble  3.1.8     ✔ dplyr   1.0.9
    ## ✔ tidyr   1.2.0     ✔ stringr 1.4.1
    ## ✔ readr   2.1.2     ✔ forcats 0.5.2
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(rvest)
```

    ## 
    ## Attaching package: 'rvest'
    ## 
    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
library(stringr)
```

## APIs

#### 1. Using the NCBI API, look for papers that show up under the term “sars-cov-2 trial vaccine.”

Get count.

``` r
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2+trial+vaccine")
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/div[1]")
counts <- as.character(counts)
stringr::str_extract(counts, "[0-9,]+")
```

    ## [1] "4,009"

Get pubmed ids.

``` r
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(
    db = "pubmed",
    term = "sars-cov-2 trial vaccine",
    retmax = 250
  )
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

There are 4,009 papers.

#### 2. Using the list of pubmed ids you retrieved, download each papers’ details using the query parameter `rettype = abstract`. If you get more than 250 ids, just keep the first 250.

``` r
ids <- as.character(ids)
ids <- stringr::str_extract_all(ids, "<Id>[[:digit:]]+</Id>")[[1]]
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
```

Grab publications with Pubmed ID list.

``` r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db      = "pubmed",
    id      = paste(ids, collapse = ","),
    rettype = "abstract"
    )
)
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

#### 3. Create a dataset

Use the `xml2::xml_children()` function to keep one element per id

``` r
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)
```

Extract abstracts

``` r
abstracts <- str_extract(pub_char_list, "<Abstract>(.|\\n)*</Abstract>")
abstracts <- str_remove_all(abstracts, "</?Abstract[^<>]*>")
abstracts <- str_replace_all(abstracts, "[[:space:]]+", " ")
```

Extract titles

``` r
titles <- str_extract(pub_char_list, "<ArticleTitle>.*</ArticleTitle>")
titles <- str_remove_all(titles, "</?ArticleTitle[^<>]*>")
```

Names of the journals

``` r
journals <- str_extract(pub_char_list, "<Title>.*</Title>")
journals <- str_remove_all(journals, "</?Title[^<>]*>")
```

Publication dates

``` r
dates <- str_extract(pub_char_list, "<PubDate>(.|\\n)*</PubDate>")
dates <- str_remove_all(dates, "</?PubDate[^<>]*>")
dates <- str_replace_all(dates, "[[:space:]]+"," ")
```

``` r
data_table <- data.frame(
  PubMedID = ids,
  Title = titles,
  Abstract = abstracts,
  Journal = journals,
  PubDate = dates
)
knitr::kable(data_table[1:5,])
```

| PubMedID | Title                                                                                                                                                                                                                 | Abstract                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | Journal                             | PubDate                                            |
|:---------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------|:---------------------------------------------------|
| 36328879 | Safety, tolerability and immunogenicity of Biological E’s CORBEVAX™ vaccine in children and adolescents: A prospective, randomised, double-blind, placebo controlled, phase-2/3 study.                                | After establishing safety and immunogenicity of Biological-E’s CORBEVAX™ vaccine in adult population (18-80 years) in Phase 1-3 studies, vaccine is further tested in children and adolescents in this study. This is a phase-2/3 prospective, randomised, double-blind, placebo-controlled study evaluating safety, reactogenicity, tolerability and immunogenicity of CORBEVAX™ vaccine in children and adolescents of either gender between \<18 to ≥12 years of age in Phase-2 and \<18 to ≥5 years of age in Phase-Phase-2/Phase-3 with placebo as a control. This study has two age sub-groups; subgroup-1 with subjects \<18 to ≥12 years of age and subgroup-2 with subjects \<12 to ≥5 years of age. In both sub groups, eligible subjects (SARS-CoV-2 RT-PCR negative and seronegative at baseline) were randomized to receive either CORBEVAX™ vaccine or Placebo in 3:1 ratio. The safety profile of CORBEVAX™ vaccine in both pediatric cohorts was comparable to the placebo-control group. Majority of reported adverse events (AEs) were mild in nature. No severe or serious-AEs, medically attended AEs (MAAEs) or AEs of special interest (AESI) were reported during the study period and all reported AEs resolved without any sequelae. In both pediatric age groups, CORBEVAX™ vaccinated subjects showed significant improvement in humoral immune-responses in terms of anti-RBD-IgG concentrations, anti-RBD-IgG1 titers, neutralizing-antibody (nAb)-titers against Ancestral-Wuhan and Delta-strains. Significantly high interferon-gamma immune- response (cellular) was elicited by CORBEVAX™ vaccinated subjects with minimal effect on IL-4 cytokine secretion. The safety profile of CORBEVAX™ vaccine in \<18 to ≥5 years’ children and adolescents was found to be safe and tolerable. Significant increase in anti-RBD-IgG and nAb-titers and IFN-gamma immune-responses were observed post-vaccination in both pediatric age sub-groups. The nAb titers observed in both the pediatric age cohorts were non-inferior to the adult cohort (BECT069 study) in terms of ratio of the GMT’s of both the cohorts. This study shows that CORBEVAX™ vaccine is highly immunogenic and can be safely administered to pediatric population as young as 5 years old. The study was prospectively registered with clinical trial registry of India- CTRI/2021/10/037066. <CopyrightInformation>Copyright © 2022 Elsevier Ltd. All rights reserved.</CopyrightInformation>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Vaccine                             | <Year>2022</Year> <Month>Oct</Month> <Day>31</Day> |
| 36328399 | Immunogenicity and safety of a three-dose SARS-CoV-2 vaccination strategy in patients with immune-mediated inflammatory diseases on immunosuppressive therapy.                                                        | Humoral vaccine responses to SARS-CoV-2 vaccines are impaired and short lasting in patients with immune-mediated inflammatory diseases (IMID) following two vaccine doses. To protect these vulnerable patients against severe COVID-19 disease, a three-dose primary vaccination strategy has been implemented in many countries. The aim of this study was to evaluate humoral response and safety of primary vaccination with three doses in patients with IMID. Patients with IMID on immunosuppressive therapy and healthy controls receiving three-dose and two-dose primary SARS-CoV-2 vaccination, respectively, were included in this prospective observational cohort study. Anti-Spike antibodies were assessed 2-4 weeks, and 12 weeks following each dose. The main outcome was anti-Spike antibody levels 2-4 weeks following three doses in patients with IMID and two doses in controls. Additional outcomes were the antibody decline rate and adverse events. 1100 patients and 303 controls were included. Following three-dose vaccination, patients achieved median (IQR) antibody levels of 5720 BAU/mL (2138-8732) compared with 4495 (1591-6639) in controls receiving two doses, p=0.27. Anti-Spike antibody levels increased with median 1932 BAU/mL (IQR 150-4978) after the third dose. The interval between the vaccine doses and vaccination with mRNA-1273 or a combination of vaccines were associated with antibody levels following the third dose. Antibody levels had a slower decline-rate following the third than the second vaccine dose, p\<0.001. Adverse events were reported by 464 (47%) patients and by 196 (78%) controls. Disease flares were reported by 70 (7%) patients. This study shows that additional vaccine doses to patients with IMID contribute to strong and sustained immune-responses comparable to healthy persons vaccinated twice, and supports repeated vaccination of patients with IMID. NCT04798625. <CopyrightInformation>© Author(s) (or their employer(s)) 2022. Re-use permitted under CC BY-NC. No commercial re-use. See rights and permissions. Published by BMJ.</CopyrightInformation>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | RMD open                            | <Year>2022</Year> <Month>Nov</Month>               |
| 36327352 | S-217622, a SARS-CoV-2 main protease inhibitor, decreases viral load and ameliorates COVID-19 severity in hamsters.                                                                                                   | In parallel with vaccination, oral antiviral agents are highly anticipated to act as countermeasures for the treatment of the coronavirus disease 2019 (COVID-19) pandemic caused by severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2). Oral antiviral medication demands not only high antiviral activity, but also target specificity, favorable oral bioavailability, and high metabolic stability. Although a large number of compounds have been identified as potential inhibitors of SARS-CoV-2 infection in vitro, few have proven to be effective in vivo. Here, we show that oral administration of S-217622 (ensitrelvir), an inhibitor of SARS-CoV-2 main protease (M<sup>pro</sup>, also known as 3C-like protease), decreases viral load and ameliorates disease severity in SARS-CoV-2-infected hamsters. S-217622 inhibited viral proliferation at low nanomolar to sub-micromolar concentrations in cells. Oral administration of S-217622 demonstrated favorable pharmacokinetic properties and accelerated recovery from acute SARS-CoV-2 infection in hamster recipients. Moreover, S-217622 exerted antiviral activity against SARS-CoV-2 variants of concern (VOCs), including the highly pathogenic Delta variant and the recently emerged Omicron BA.5 and BA.2.75 variants. Overall, our study provides evidence that S-217622, an antiviral agent that is under evaluation in a phase 3 clinical trial (clinical trial registration no. jRCT2031210350), possesses remarkable antiviral potency and efficacy against SARS-CoV-2 and is a prospective oral therapeutic option for COVID-19.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Science translational medicine      | <Year>2022</Year> <Month>Nov</Month> <Day>03</Day> |
| 36322837 | Covid-19 Vaccine Protection among Children and Adolescents in Qatar.                                                                                                                                                  | The BNT162b2 vaccine against coronavirus disease 2019 (Covid-19) has been authorized for use in children 5 to 11 years of age and adolescents 12 to 17 years of age but in different antigen doses. We assessed the real-world effectiveness of the BNT162b2 vaccine against infection with severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2) among children and adolescents in Qatar. To compare the incidence of SARS-CoV-2 infection in the national cohort of vaccinated participants with the incidence in the national cohort of unvaccinated participants, we conducted three matched, retrospective, target-trial, cohort studies - one assessing data obtained from children 5 to 11 years of age after the B.1.1.529 (omicron) variant became prevalent and two assessing data from adolescents 12 to 17 years of age before the emergence of the omicron variant (pre-omicron study) and after the omicron variant became prevalent. Associations were estimated with the use of Cox proportional-hazards regression models. Among children, the overall effectiveness of the 10-μg primary vaccine series against infection with the omicron variant was 25.7% (95% confidence interval \[CI\], 10.0 to 38.6). Effectiveness was highest (49.6%; 95% CI, 28.5 to 64.5) right after receipt of the second dose but waned rapidly thereafter and was negligible after 3 months. Effectiveness was 46.3% (95% CI, 21.5 to 63.3) among children 5 to 7 years of age and 16.6% (95% CI, -4.2 to 33.2) among those 8 to 11 years of age. Among adolescents, the overall effectiveness of the 30-μg primary vaccine series against infection with the omicron variant was 30.6% (95% CI, 26.9 to 34.1), but many adolescents had been vaccinated months earlier. Effectiveness waned over time since receipt of the second dose. Effectiveness was 35.6% (95% CI, 31.2 to 39.6) among adolescents 12 to 14 years of age and 20.9% (95% CI, 13.8 to 27.4) among those 15 to 17 years of age. In the pre-omicron study, the overall effectiveness of the 30-μg primary vaccine series against SARS-CoV-2 infection among adolescents was 87.6% (95% CI, 84.0 to 90.4) and waned relatively slowly after receipt of the second dose. Vaccination in children was associated with modest, rapidly waning protection against omicron infection. Vaccination in adolescents was associated with stronger, more durable protection, perhaps because of the larger antigen dose. (Funded by Weill Cornell Medicine-Qatar and others.). <CopyrightInformation>Copyright © 2022 Massachusetts Medical Society.</CopyrightInformation>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | The New England journal of medicine | <Year>2022</Year> <Month>Nov</Month> <Day>02</Day> |
| 36320825 | Withholding methotrexate after vaccination with ChAdOx1 nCov19 in patients with rheumatoid or psoriatic arthritis in India (MIVAC I and II): results of two, parallel, assessor-masked, randomised controlled trials. | There is a necessity for an optimal COVID-19 vaccination strategy for vulnerable population groups, including people with autoimmune inflammatory arthritis on immunosuppressants such as methotrexate, which inhibit vaccine-induced immunity against SARS-CoV-2. Thus, we aimed to assess the effects of withholding methotrexate for 2 weeks after each dose of ChAdOx1 nCov-19 (Oxford-AstraZeneca) vaccine (MIVAC I) or only after the second dose of vaccine (MIVAC II) compared with continuation of methotrexate, in terms of post-vaccination antibody titres and disease flare rates. MIVAC I and II were two parallel, independent, assessor-masked, randomised trials. The trials were done at a single centre (Dr Shenoy’s Centre for Arthritis and Rheumatism Excellence; Kochi, India) in people with either rheumatoid arthritis or psoriatic arthritis with stable disease activity, who had been on a fixed dose of methotrexate for the preceding 6 weeks. Those with previous COVID-19 or who were positive for anti-SARS-CoV-2 nucleocapsid antibodies were excluded from the trials. People on high-dose corticosteroids and rituximab were also excluded, whereas other disease-modifying antirheumatic drugs were allowed. In MIVAC I, participants were randomly assigned (1:1) to stop methotrexate treatment for 2 weeks after each vaccine dose or to continue methotrexate treatment. In MIVAC II, participants who had continued methotrexate during the first dose of vaccine were randomly assigned (1:1) to withhold methotrexate for 2 weeks after the second dose of vaccine or to continue to take methotrexate. The treating physician was masked to the group assignments. The primary outcome for both MIVAC I and MIVAC II was the titre (absolute value) of anti-receptor binding domain (RBD) antibody measured 4 weeks after the second dose of vaccine. All analyses were done per protocol. The trials were registered with the Clinical Trials Registry- India, number CTRI/2021/07/034639 (MIVAC I) and CTRI/2021/07/035307 (MIVAC II). Between July 6 and Dec 15, 2021, participants were recruited to the trials. In MIVAC I, 250 participants were randomly assigned and 158 completed the study as per the protocol (80 in the methotrexate hold group and 78 in the control group; 148 \[94%\] were women and 10 \[6%\] were men). The median post-vaccination antibody titres in the methotrexate hold group were significantly higher compared with the control group (2484·0 IU/mL, IQR 1050·0-4388·8 <i>vs</i> 1147·5 IU/mL, 433·5-2360·3; p=0·0014). In MIVAC II, 178 participants were randomly assigned and 157 completed the study per protocol (76 in the methotrexate hold group and 81 in the control group; 135 \[86%\] were women and 22 \[14%\] were men). The methotrexate hold group had higher post-vaccination antibody titres compared with the control group (2553·5 IU/ml, IQR 1792·5-4823·8 <i>vs</i> 990·5, 356·1-2252·5; p\<0·0001). There were no reports of any serious adverse events during the trial period. Withholding methotrexate after both ChAdOx1 nCov-19 vaccine doses and after only the second dose led to higher anti-RBD antibody titres compared with continuation of methotrexate. However, withholding methotrexate only after the second vaccine dose resulted in a similar humoral response to holding methotrexate after both vaccine doses, without an increased risk of arthritis flares. Hence, interruption of methotrexate during the second dose of ChAdOx1 nCov-19 vaccine appears to be a safe and effective strategy to improve the antibody response in patients with rheumatoid or psoriatic arthritis. Indian Rheumatology Association. <CopyrightInformation>© 2022 Elsevier Ltd. All rights reserved.</CopyrightInformation> | The Lancet. Rheumatology            | <Year>2022</Year> <Month>Nov</Month>               |

## Text Mining

``` r
library(tidytext)
library(dplyr)
library(data.table)
```

    ## 
    ## Attaching package: 'data.table'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     between, first, last

    ## The following object is masked from 'package:purrr':
    ## 
    ##     transpose

``` r
library(ggplot2)
```

Download dataset

``` r
if (!file.exists("pubmed.csv"))
  download.file(
    url = "https://raw.githubusercontent.com/USCbiostats/data-science-data/master/03_pubmed/pubmed.csv",
    destfile = "pubmed.csv",
    method   = "libcurl",
    timeout  = 60
    )
pubmed <- fread("pubmed.csv")
```

#### 1. Tokenize the abstracts and count the number of each token. Do you see anything interesting? Does removing stop words change what tokens appear as the most frequent? What are the 5 most common tokens for each search term after removing stopwords?

Tokenize the abstracts and count the number of each token.

``` r
pubmed %>%
  unnest_tokens(token, abstract) %>%
  count(token, sort = TRUE) %>%
  top_n(5, n) %>%
  ggplot(aes(n, fct_reorder(token, n))) +
    geom_col() +
    labs(x = "Frequency", y = "Token", title = "Top 5 most common tokens")
```

![](Assignment_03_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

Among the top 5 frequent words, all of them are stop words (the, of,
and, in, to).

Let’s try after removing stop words.

``` r
pubmed %>%
  unnest_tokens(token, abstract) %>%
  anti_join(stop_words, by = c("token" = "word")) %>%
  count(token, sort = TRUE) %>%
  filter(!grepl(pattern = "^[0-9]+$", x = token)) %>%
  top_n(5, n) %>%
  ggplot(aes(n, fct_reorder(token, n))) +
  labs(x = "Frequency", y = "Token", title = "Top 5 most common tokens") +
  geom_col()
```

![](Assignment_03_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

After removing stop words, the 5 most frequent words are covid,
patients, cancer, prostate and disease.

#### 2. Tokenize the abstracts into bigrams. Find the 10 most common bigram and visualize them with ggplot2.

``` r
pubmed %>%
  unnest_ngrams(bigram, abstract, n = 2) %>%
  count(bigram, sort = TRUE) %>%
  top_n(10, n) %>%
  ggplot(aes(n, fct_reorder(bigram, n))) +
    geom_col() +
    labs(x = "Frequency", y = "Token", title = "Top 10 most common bigrams")
```

![](Assignment_03_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

The 10 most common bigrams are: *covid 19*, *of the*, *in the*,
*prostate cancer*, *pre eclampsia*, *patients with*, *of covid*. *and
the*, and *of prostate*.

#### 3. Calculate the TF-IDF value for each word-search term combination. (here you want the search term to be the “document”) What are the 5 tokens from each search term with the highest TF-IDF value? How are the results different from the answers you got in question 1?

``` r
TFIDF <- pubmed %>%
  unnest_tokens(token, abstract) %>%
  count(token, term) %>%
  bind_tf_idf(token, term, n) %>%
  arrange(desc(tf_idf))
knitr::kable(TFIDF[1:5,])
```

| token        | term            |    n |        tf |      idf |    tf_idf |
|:-------------|:----------------|-----:|----------:|---------:|----------:|
| covid        | covid           | 7275 | 0.0371050 | 1.609438 | 0.0597183 |
| prostate     | prostate cancer | 3832 | 0.0311890 | 1.609438 | 0.0501967 |
| eclampsia    | preeclampsia    | 2005 | 0.0142784 | 1.609438 | 0.0229802 |
| preeclampsia | preeclampsia    | 1863 | 0.0132672 | 1.609438 | 0.0213527 |
| meningitis   | meningitis      |  429 | 0.0091942 | 1.609438 | 0.0147974 |

The Top5 most frequent words are covid, prostate, eclampsia,
preeclampsia and meningitis. The result is different from Question 1.
The results of Q1 and Q2 both include covid and prostate. The result of
Q3 has many terminologies, by contrast, the result of Q1 is more general
(patients, cancer, and disease). I think the result of Q3 may be more
helpful.
