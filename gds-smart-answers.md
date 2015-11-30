# Who we are

James Mead and Chris Roos

Go Free Range Ltd

---

# What are Smart Answers

* TODO: Demonstrate real smart answer

---

# What we were asked to do

* Reduce maintenance cost of Smart Answers
  * Developer time
  * Content Team time
* Ideally end up with a system that makes it easy for Content Team to edit content

---

# How we approached the problem

---

## Four implementations

* Simple Smart Answers
* Calculators
* Smartdown Smart Answers
* Ruby Smart Answers

^
We investigated the four ways of building them

---

## Simple Smart Answers

* Created in Publisher
* Stored in Content store - TODO: Where are these stored?
* Rendered through Frontend

---

## Simple Smart Answers

* Advantages
  * Familiar publishing workflow for content editors

* Disadvantages
  * Very limited functionality
  * Multiple representations of Smart Answer
  * UI divergence

^
* Very limited functionality
  * Only decision trees
  * Routing can only depend on latest response
  * No calculations
  * No country questions
* Two representations of Smart Answer
  * Code in Publisher app
  * Code in Frontend app
  * TODO: Are there 3 places? GDS adaptors Gem? Content API?
* Code quality
  * Javascript in Publisher - no unit tests? TODO: Investigate this.
  * Inappropriate use of inheritance in Publisher

---

## Calculators

* Child Benefit tax calculator
* Separate Rails app
* All questions on a single page
* Dynamic number of questions

---

## Calculators

* Advantages
  * Multiple questions per page?
* Disadvantages
  * Maintenance cost of a separate Rails app
  * UI divergence

^
* Multiple questions per page?
  * Although we're not convinced this is necessarily an advantage.
  * Analytics data for Pay leave for parents suggests that switching to a single question per page hasn't made completion rates any worse.

---

## Smartdown Smart Answers

### Overview

* Flows defined using Smartdown library
* User interactions via Smart Answers app

^
* Flows defined using Smartdown library
  * Uses Govspeak for question and outcome pages
  * Uses nested bullet points to describe routing/flow logic
* User interactions via Smart Answers app
  * Adapter layer

---

## Smartdown Smart Answers

### Advantages

* Familiar Govspeak-like templates
* More obviously editable by content editors
* Smartdown template is just text
* Multiple questions per page?

^
* Smartdown template closely resembles rendered page
* All page content in a single place
  * Particularly when compared to original Ruby Smart Answers
* Smartdown template is just text
  * Storing the rules in text makes it possible to store them as content - put them through publishing workflow
* Multiple questions per page?
  * Not obviously an advantage

---

## Smartdown Smart Answers

### Disadvantages

* Unfamiliar custom templating language
* Unfamiliar syntax describing logic flow
* Missing features
* Complexity of parser

^
* Unfamiliar custom templating language
  * Re-inventing, e.g. $IF/$ELSE conditionals, Partials, Interpolation
* Unfamiliar syntax describing logic flow
  * Feels as if we’re re-inventing Ruby (we could use `if/else` or `case`)
* Missing features
  * Country dropdowns. TODO: Double check this
  * More development required
* Complexity of parser
  * Maintenance overhead - Mixture of Parslet rules and parsing using regular expressions

---

## Ruby Smart Answers

### Advantages

* Mostly plain old Ruby
* Fully featured

---

## Ruby Smart Answers

### Disadvantages

* Possibly too flexible?
* Very non-standard Rails app
* Confusing DSL
* Page content scattered
* Single question per page?

^
* Fully featured
  * Supports existing complicated Smart Answers
* Possibly too flexible?
  * Easy to do the same thing in different ways leading to a lack of consistency across Smart Answers
* Very non-standard Rails app
  * Handful of standard controller actions but vast majority of the code doesn't follow the Rails way
* Confusing DSL
  * Multiple ways to do the same thing
  * Predicate code
    * e.g. order of arguments in next_node_if
    * Re-inventing Ruby logic
  * Non-obvious order of execution
* Page content scattered
  * Pages made up of multiple phrases from different places in YAML file assembled by separate chunk of Ruby code - makes it harder to work out what the page will look like
* Single question per page?
  * No support for multiple questions per page
  * No support for dynamic number of questions

---

## New Smart Answer format?

* Easy for content people to edit
* Works with publishing workflow

^
* There was some initial discussion about creating a new format and building a user interface that would allow content editors to build Smart Answers using more familiar publishing tools/workflow.

---

# What we decided to do

* No new Smart Answer format
* Reduce number of implementations
* Improve Ruby Smart Answers

^
* No new Smart Answer format
  * Big bang approach - have to be feature complete before we can migrate all existing Smart Answers to it
* Reduce number of implementations
  * Maintenance overhead of having 4 ways to do the same thing
  * We think it’s better to rationalise the four existing implementations into one and then iteratively improve that
* Improve Ruby Smart Answers
  * Majority of existing Smart Answers are already in this format
  * Better separation of concerns e.g. presentation logic, flow logic & policy logic all mixed up together
    * This problem wasn't unique to Ruby Smart Answers
  * Include some of the features of Smartdown that we liked
  * ADR001 explains why we chose Ruby over Smartdown

---

# What we’ve done

---

## Improved Fact check process

^
* Discuss old v2 workflow
* Git branches
* Deploy to Heroku - using startup_heroku.sh script

---

## Comprehensive regression tests

^
* Relatively non-implementation specific
* Helped give us confidence that we haven't broken anything when making more fundamental changes to the code
* Capture rendered pages (questions and outcomes) for later comparison
* We hope they'll be replaced by tests around better factored Smart Answers
* Can be painful to maintain - slow to run and the diffs can be hard to analyse
* Jenkins instance

---

## ERB templates

^
* Move from I18n YAML files to ERB templates for questions and outcomes
* More like Smartdown - all content in a single place, one file per page
* More like standard Rails app - familiar to new developers

---

## Separating concerns

^
* Question definitions & routing logic should only be in Flow class
* Policy & validation logic should only be in Calculator/Policy class
* Presentation logic should only be in ERB templates or helper methods
* Content should only be in ERB templates
* We've only done this in a few places

---

## Domain modelling

^
* DateRange (leap year handling, number of days in period)
* TaxYear (5th/6th April mentioned in a number of places)
* "Calculate your employee's statutory sick pay"
  * Period of Incapacity for Work (PIW) not explicitly modeled
  * When linked period of sickness functionality added, unnecessary duplicate logic added
* We've only done this in a few places

---

## Converting Smartdown flows to Ruby

^
* Convert pay-leave-for-parents to one question per page first, rather than adding support for multiple questions per page in Ruby
* Regression tests gave us confidence that we hadn't broken anything
* No longer using Smartdown
* We've removed all Smartdown specific code from Smart Answers app, and deprecated the Smartdown repository

---

## Simplifying the flow DSL

^
* One way of specifying next node rules.
  * Removed predicate functionality
  * Removed multiple choice shortcut
  * The resulting next_node blocks can be slightly more verbose but we agreed to focus on consistency first and to make it less verbose later.
* Parsing responses, e.g. Date questions now give you a date object

---

## Less surprising code

^
* Avoid `eval`ing flow logic
  * Individual Flow subclasses allowed us to get code coverage which helped when generating regression tests
* Failing fast rather than useless defaults
  * Start node and Question titles used to fallback to humanised key
  * Undefined State variables defaulted to nil, which made it hard to find bugs due to typos

---

## Make things more Rails like

^
* Using ActionView::Base to render landing page, question and outcome templates
* Using Rails helper methods (e.g. number_to_currency)
* Using helper methods included in the View rather than elsewhere

---

## New Smart Answer

^
* HMRC's "Calculate your part-year profits to finalise your tax credits"
* Visualisation to help illustrate/describe how the various dates interacted
* Highlighted some of the pain with the current process
* Logic doc contained the rules but not necessarily the reasons behind them
* Started with a simple scenario and then enhanced it to support more complicated scenarios
* New testing approaches that we think are better and have allowed us to avoid adding regression tests for this Smart Answer
* First time we applied our ideas about separating concerns and domain modelling.

---

## Documentation

^
* We've written quite a lot of documentation and it's all accessible from the README in the repository.

---

# What we would do next

^
* A selection of things we'd probably choose to work on next, but not necessarily in this order

---

## Improve FCO Smart Answers

^
* These are some of the most painful Smart Answers due to the number of countries/territories and the frequency of changes to the rules
* We might investigate country specific outcome pages

---

## Process improvements

^
* Smart Answers aren’t just content - each one is a little app
* Earlier developer involvement
* Process is very waterfally
  * Logic docs
    * Up front design
    * Misses the “why” behind the rules
  * Anything we can do to make it more incremental/iterative would be an improvement
    * Sit/work closely with policy people in the department?
* Build a very simple version of a new Smart Answer
   * Handle a single happy path
   * Describes the cases it doesn’t handle
   * Get it deployed and get feedback
   * Incrementally cover more scenarios
   * Easier for a developer to understand if built incrementally rather than trying to understand the entire flow in one go

---

## Separate concerns of remaining flows

^
* Add better unit/integration tests, similar to those in part-year-profit-tax-credits
* Remove regression tests

---

## Improve content team workflow

^
* Currently:
  * Content editor uses GitHub to fork Smart Answers and change ERB templates
  * Opens PR from forked repo
  * We regenerate regression test artefacts and open new PR
  * We deploy the changes to Heroku for fact check
* We can imagine streamlining this process

---

## Move calculators

^
* Shouldn't be too hard assuming we can move to one question per page
* Reduce amount of code we have to support
* Although it hasn't needed changing for a long time, we think converting this, and reducing the number of apps we have to support should mean that we prioritise this.

---

## Render Simple Smart Answers using Smart Answers app

^
* Avoid existing and future UI divergence
* It shouldn't be too hard given that the Smart Answers app already has access to the Content API.
* Reduce amount of code we have to support

---

## Convert Simple Smart Answers to Ruby

^
* Reduce amount of code we have to support
* Allows us to express the intent of the policy logic better
* Not right now as it takes control away from content team

---

## Further DSL improvements

^
* Automatically instantiate calculator/policy object (and remove next_node_calculation blocks)
* Automatically save the response to a question
* Remove calculate blocks?
* Remove precalculate blocks?
* Automatically determine permitted next nodes
* New question types, e.g. yes/no question type, integer question type, float question type

---

## Improve ERB templates

^
* Avoid using content_for for body content
* Reduce number of content_for blocks in question pages

---

## Trello

^
* We have a Trello board with lots more information. Ask if you want us to give you access. Or can we make it public?

---

# Conclusion

---

## Recap: What we were asked to do

* Reduce maintenance cost of Smart Answers
  * Developer time
  * Content Team time
* Ideally end up with a system that makes it easy for Content Team to edit content

^
* Hopefully we're leaving the code in a better state
  * Hopefully reduced maintenance cost in terms of developer
* There's still lots to do
  * Particularly around content editors ability to edit content
* If you have any questions after we've gone then please do get in touch - we're keen to see this project succeed

---

## Business as usual

^
* We'd like to encourage people to continue to make small improvements to the code whenever they have to make business as usual changes
* Hopefully the regression tests give people confidence to make such changes

---

## Examples in flat content

TODO: Insert screenshot of examples table

^ * https://www.gov.uk/guidance/statutory-sick-pay-manually-calculate-your-employees-payments#calculate-ssp-including-rates
* Examples table could be generated by policy object code or used to check that code

---

## Policy as code

TODO: Insert screenshot of tweet/gist

^
* People don’t like working on Smart Answers - we think it is/can be really interesting
* We've enjoyed working on it
* https://twitter.com/richardjpope/status/620848391006867456
* Richard's gist - https://gist.github.com/memespring/55f8529f1d9e6dd76632
* Tom S's gist - https://gist.github.com/tomstuart/7249ad30e56c45d227da

---

## Pub?

^
* We'll be at the Princess Louise from 5:00