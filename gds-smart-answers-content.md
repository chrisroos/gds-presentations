# What we were asked to do

* Reduce maintenance cost of Smart Answers
  * Developer time
  * Content Team time
* Ideally end up with a system that makes it easy for Content Team to edit content

---

# What we've done

---

## Removed Smartdown

^
* No multiple questions per page - not obvious that we'll want to add it back

---

## Domain modelling

^
* TaxYear (5th/6th April mentioned in a number of places)
* "Calculate your employee's statutory sick pay"
  * Period of Incapacity for Work (PIW) not explicitly modeled
  * When linked period of sickness functionality added, unnecessary duplicate logic added
* We've only done this in a few places

---

## Improved Factcheck process

^
* Deploy to Heroku

---

## Editing ERB templates

^
* Directory structure

---

## New HMRC Smart Answer

^
* HMRC's "Calculate your part-year profits to finalise your tax credits"
* Visualisation to help illustrate/describe how the various dates interacted
* Highlighted some of the pain with the current process
* Logic doc contained the rules but not necessarily the reasons behind them
* Started with a simple scenario and then enhanced it to support more complicated scenarios
* New testing approaches that we think are better and have allowed us to avoid adding regression tests for this Smart Answer
* First time we applied our ideas about separating concerns and domain modelling.

---

# What we'd do next

## Improve FCO Smart Answers

^
* Sorry!
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

## Improve content team workflow

^
* Currently:
  * Content editor uses GitHub to fork Smart Answers and change ERB templates
  * Opens PR from forked repo
  * We regenerate regression test artefacts and open new PR
  * We deploy the changes to Heroku for fact check
* We can imagine streamlining this process

---

## Convert Simple Smart Answers to Ruby

^
* Reduce amount of code we have to support
* Allows us to express the intent of the policy logic better
* Not right now as it takes control away from content team
* Also good to bring the calculators app into Smart Answers

---

# Conclusion

---

## Recap: What we were asked to do

* Reduce maintenance cost of Smart Answers
  * Developer time
  * Content Team time
* Ideally end up with a system that makes it easy for Content Team to edit content

^
* Mostly focussed on reducing the amount of developer time required to maintain Smart Answers
* Hopefully that has a knock on benefit to the content team
* Publishing workflow for Smart Answers is still someway off
* If you have any questions after we've gone then please do get in touch - we're keen to see this project succeed
