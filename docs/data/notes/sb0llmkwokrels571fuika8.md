
First try using NLTK: https://www.nltk.org/data.html

```python
raw_text = """
Community engagement insights on immigration issues
Political Survey Interview
    •    Gilbert from Ruger Arizona Engagement conducting nonpartisan community opinion survey
    •    Three structured questions about political issues and party performance
    •    Information collected for government review of community priorities
Key Issues Identified
    •    Immigration identified as top priority issue
    ?    Currently “really big” concern
    ?    Needs immediate attention over housing, education, child support
    •    Political dysfunction
    ?    “Too much fighting” between parties
    ?    No compromise or control
    ?    Focus on immigration and birth control debates
Immigration Policy Position
    •    Nuanced approach needed for different immigrant categories:
    1    Illegal border crossers should be deported
    2    Working immigrants with permits should receive support
    3    Pregnant mothers and families with US-born children need help
    4    Long-term residents (30+ years) with clean records shouldn’t be separated from families
    •    Criminal immigrants should be removed immediately regardless of ethnicity
    •    Economic necessity argument
    ?    Agricultural work going undone (watermelon example)
    ?    Food waste and higher prices without immigrant labor
Recommendations for Political Leaders
    •    Direct honest conversation needed between parties
    •    Sit down and negotiate real compromises
    •    Focus on practical solutions over political fighting
    •    Distinguish between working families and criminals
    •    Address root causes (mentioned family structure statistics)
    •    Stop “telling each other off” and find organized solutions
    """

clean_text = re.sub(r"[•?]","",raw_text)
clean_text = re.sub(r"^\s*\d+\s+", "", clean_text, flags=re.MULTILINE)
clean_text = "\n".join(line.strip() for line in clean_text.splitlines() if line.strip())

sia = SentimentIntensityAnalyzer()
print(sia.polarity_scores(clean_text))
```

```
{'neg': 0.113, 'neu': 0.74, 'pos': 0.147, 'compound': 0.6908 
```

Tried a couple other summary notes samples. The issue is that the neutrality score is too high because the AI notetaker is designed to neutralize and professionalize summary content.

Want to try something like this with sentiment scores for issue keywords, but need to look into documentation on how to dictionary the keywords. Can I have keywords in multiple issues? Or are they mutually exclusive. How do I choose good keywords? Looking to read through more of the next iteration of voice to text over AI summary

```python
ISSUE_KEYWORDS = {
    "immigration": {
        "single": {"immigration","border","asylum","deport","migrant","cartel","coyote","wall","daca"},
        "phrases": {("border","cross"),("family","separation")}
    },
    "housing": {
        "single": {"housing","rent","eviction","landlord","mortgage","zoning","homeless"},
        "phrases": {("affordable","housing")}
    },
    "education": {
        "single": {"school","education","teacher","curriculum","student","tuition"},
        "phrases": {("school","funding")}
    },
    "economy": {
        "single": {"job","wage","inflation","price","economy","tax","groceries"},
        "phrases": {("cost","living")}
    },
    "childcare": {
        "single": {"childcare","mother","father","parent","parents","","child","children"},
        "phrases": {("single","parents"),("single","mother"),("single","parents"),("single","income")}
    },
    "trump": {
        "single": {"trump","vance","president","vice","hegeseth","","rfk","bondi","donald"},
        "phrases": {("white","house"),("donald","trump"),("j","d","vance"),("presidential","power")}
    },
    "republican party": {
        "single": {"republican","trump","conservative","gop","rhino","hegeseth","rfk","bondi","donald"},
        "phrases": {("white","house"),("republican","party"),("j","d","vance"),("presidential","power")}
    },
    "democratic party": {
        "single": {"democrat","democratic","liberal","progressive","biden","harris","rfk","bondi","donald"},
        "phrases": {("white","house"),("democratic","party"),("j","vance"),("presidential","power")}
    }
}

def tag_issues(lemmas):
    tags = set()
    lemma_set = set(lemmas)
    for issue, kws in ISSUE_KEYWORDS.items():
        if lemma_set & kws["single"]:
            tags.add(issue)
            continue
        # phrase match: look for both tokens present (cheap proxy)
        for a,b in kws["phrases"]:
            if a in lemma_set and b in lemma_set:
                tags.add(issue); break
    return sorted(tags)

```

All in all this library is super easy to use, but I suspect harder to use well. Easy to understand maybe.