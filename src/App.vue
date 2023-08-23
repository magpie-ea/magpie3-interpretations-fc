<template>
  <Experiment title="magpie demo">
    <InstructionScreen :title="'Welcome'">
      <p>Thank you for participating in this experiment! Your data is anonymous and will only be used for research purposes by cognitive scientists from the university of Tübingen. 
        Your participation is voluntary and you may return the study at any point without adverse consequences.</p>
      There will be <strong> {{n_trials}} trials</strong> in this experiment. The experiment will take about 2 minutes.<br>

      In each trial, you will read a short description of a situation and answer a question about the situation by selecting an answer.

      <p>Please decide which of the following animals is your favorite (if you had to choose): sloth, ant-eater, koala bear. 
        Once you have decided, please press the button below to start the experiment.</p> 
    </InstructionScreen> <!-- last bit is an anttention check against LLMs -->

    <template v-for="(trial, i) in trialData">
      <Screen>

        <Slide>
          <p style="color:#A9A9A9">Please read the following conversation.</p>

          <p> 
            {{ trial.context.replace(trial.speaker.concat(" replies:"), "\n\n".concat(trial.speaker.concat(" replies:"))) }} 
            <strong>'{{ trial.trigger.charAt(0).toUpperCase() + trial.trigger.slice(1)}}'<br/>
             <br/> 
            </strong>
          </p>

          <MultipleChoiceScreen
            :question="trial.fc_question"
            :options="trial.options_list"
          />

          <Record :data="{
          'condition': conditions[i],
          'item_id': trial.item_id,
          'options_order': trial.options_order.join('|'),
          'options_list': trial.options_list.join('|'),
          'trial_nr': i + 1 }" />
        </Slide>

      </Screen>

    </template>

    <TextareaScreen 
      question="Please type in your favorite animal which you picked before the start of the study in lower case in the text area below. Please don't type anything else."
    />

    <PostTestScreen />

    <SubmitResultsScreen />
  </Experiment>
</template>

<script>
import _ from 'lodash';
import trials from '../trials/our-materials.csv';
import { PostTestScreen, TextareaScreen } from 'magpie-base';

const n_trials = 2;
// select item IDs for a participant (there are 24 items)
var trialInds = _.shuffle(_.range(1, 25)).slice(0,n_trials);
console.log(trialInds);
// sample two conditions
var conditions = _.shuffle([
  'too_little',
  'too_much',
  'irrelevant',
  'marked',
  'baseline'
]).slice(0, n_trials);
console.log(conditions);
// zip 
var trialIndsAndConditions = _.zip(trialInds, conditions);
console.log(trialIndsAndConditions);
// get trial data for that IDs and conditions
var trialData = _.filter(trials, (row) => {
  return trialIndsAndConditions.some(([itemId, condition]) => {
    return row.item_id === itemId && row.inference_type === condition;
  })
});
console.log(trialData);
// shuffle the order of FC options for each trial
trialData.forEach(t => {
    t.options_order =  _.shuffle([
      'target_prejacent', 
      'competitor_prejacent',
      'distractor_1_prejacent',
      'distractor_2_prejacent'
    ]);
    t.options_list = t.options_order.map(x => t[x]);
    t.fc_question = "Which of the following four sentences best expresses what " + t.speaker + " meant with their utterance?";
})

console.log(trialData);

export default {
    name: 'App',
    data() {
        return {
            trialData: trialData,
            conditions: conditions,
            n_trials: n_trials
        };
    },
    computed: {
        // Expose lodash to template code
        _() {
            return _;
        }
    },
    components: { TextareaScreen, PostTestScreen }
};
</script>
