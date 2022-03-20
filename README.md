# Click Bait Generator

[Datacraft](https://github.com/bbux-dev/datacraft) spec for generating Click Bait Headlines

# Click Bait Structure From:

https://inventwithpython.com/bigbookpython/project11.html

# Usage

```shell
$ pip install datacraft

$ datacraft -s click-bait-spec.json --sample-lists -i 1 --log-level warn
Are Millenials Killing the Paleo Diet Industry?
Without This Plastic Straw, Avocados Could Kill You Next Week
Big Companies Hate Her! See How This Georgia Chicken Invented a Cheaper Avocado
You Won't Believe What This California Shovel Found in Her Donut Shop
13 Gift Ideas to Give Your Paleo Diet From Michigan
14 Reasons Why Shovels Are More Interesting Than You Think (Number 2 Will Surprise You!)
This New York Doctor Didn't Think Robots Would Take Their Job. They Were Wrong.
```

# Spec Tricks

This data spec has to use some tricks to get the expected output.  

## Sample Lists CLI arg

Every time we run the spec, we would like the output to be somewhat random.  We make use of the `--sample-lists` 
command line argument to accomplish this. This makes the output random and more interesting.

## Same Key in Template
The template for the Big Companies click bate is:

`Big Companies Hate {{ OBJECT_PRONOUNS }}! See How This {{ STATES }} {{ NOUNS }} Invented a Cheaper {{ NOUNS2 }}`

Example:

`Big Companies Hate Her! See How This Georgia Chicken Invented a Cheaper Avocado`

This sentence has two Nouns that are substituted in. We could have defined two lists with the same elements. Instead,
we define a NOUNS  element with a list of Nouns to use. We then define NOUNS2 as a `ref` type that point to the NOUNS 
reference.  Since we are using the `--sampe-lists` setting for all list based suppliers, this will give a random 
value for NOUNS and NOUNS2.

Below is the Field Spec and refs to support this template:

```json
{
  "bigCompanies": {
    "type": "templated",
    "refs": ["OBJECT_PRONOUNS", "STATES", "NOUNS", "NOUNS2"],
    "data": "Big Companies Hate {{ OBJECT_PRONOUNS }}! See How This {{ STATES }} {{ NOUNS }} Invented a Cheaper {{ NOUNS2 }}"
  },
  "refs": {
    "OBJECT_PRONOUNS": ["Her", "Him", "Them"],
    "STATES": ["California", "Texas", "Florida", "New York", "Pennsylvania",
              "Illinois", "Ohio", "Georgia", "North Carolina", "Michigan"],
    "NOUNS": ["Athlete", "Clown", "Shovel", "Paleo Diet", "Doctor", "Parent",
             "Cat", "Dog", "Chicken", "Robot", "Video Game", "Avocado",
             "Plastic Straw","Serial Killer", "Telephone Psychic"],
    "NOUNS2:ref": "NOUNS"
  }
}
```

## Embedded Jinja2 Logic

The Job Automated Click Bait Template is:

`This {{ STATES }} {{ NOUNS }} Didn't Think Robots Would Take {{ PRONOUN_PAIRS[0] }} Job.`

`{{ PRONOUN_PAIRS[1] }}{%if PRONOUN_PAIRS[0] == 'Their' %} Were {% else %} Was {% endif %}Wrong.`

Example:

`This Illinois Telephone Psychic Didn't Think Robots Would Take His Job. He Was Wrong.`

Below is the spec and references to support it:

```json
{
  "jobAutomated": {
    "type": "templated",
    "refs": ["STATES", "NOUNS", "PRONOUN_PAIRS"],
    "data": "This {{ STATES }} {{ NOUNS }} Didn't Think Robots Would Take {{ PRONOUN_PAIRS[0] }} Job. {{ PRONOUN_PAIRS[1] }}{%if PRONOUN_PAIRS[0] == 'Their' %} Were {% else %} Was {% endif %}Wrong."
  },
  "refs": {
    "PRONOUN_PAIRS": [["Her", "She"], ["His", "He"], ["Their", "They"]],
    "STATES": ["California", "Texas", "Florida", "New York", "Pennsylvania",
              "Illinois", "Ohio", "Georgia", "North Carolina", "Michigan"],
    "NOUNS": ["Athlete", "Clown", "Shovel", "Paleo Diet", "Doctor", "Parent",
             "Cat", "Dog", "Chicken", "Robot", "Video Game", "Avocado",
             "Plastic Straw","Serial Killer", "Telephone Psychic"]
    }
}
```

We want the sentence to make as much sense as possible. Therefore, we define the PRONOUN_PAIRS reference as a List of 
Lists.  We then index the first element to get the possessive form and the second when we want the subject form. By 
defining them as pairs of elements, we can make sure the related ones stay together.

### Conditionals

Certain pronouns will condition the use of was or were for the final bit of the headline. We use the Jinja2 `if` 
operator to enable this conditional inclusion directly in the template string itself, i.e.:

`{{ PRONOUN_PAIRS[1] }}{%if PRONOUN_PAIRS[0] == 'Their' %} Were {% else %} Was {% endif %}Wrong.`
