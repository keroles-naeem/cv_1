CV 

Got it 👍

In GitLab CI/CD, $CI_PIPELINE_ID is just an integer like 1023. If you want to turn it into a semantic version-like format (1.0.23), you need to split the digits and format them.

For example, if 1023 → 1.0.23 (rule: take first digit as major, second as minor, last two as patch):

You can do this in your .gitlab-ci.yml with a small shell script:

variables:
  VERSION: ""

before_script:
  - export VERSION="$(echo $CI_PIPELINE_ID | sed -E 's/([0-9])([0-9])([0-9]{2})/\1.\2.\3/')"
  - echo "VERSION=$VERSION"

If you want it simpler in one line:

VERSION=$(echo $CI_PIPELINE_ID | sed -E 's/([0-9])([0-9])([0-9]{2})/\1.\2.\3/')

So for:

1023 → 1.0.23

1125 → 1.1.25

2345 → 2.3.45



---

👉 Question: Do you always want it to be [first].[second].[last two digits] or should it be [first two].[next].[last]?
This way I’ll adapt the regex for you.

