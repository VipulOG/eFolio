<script lang="ts">
  import type { CssClasses } from '@skeletonlabs/skeleton';
  import { createEventDispatcher, setContext } from 'svelte';
  import { type Writable, writable } from 'svelte/store';
  import type { StepperButton, StepperEvent, StepperState } from './types.js';

  // Event Dispatcher
  const dispatch = createEventDispatcher<StepperEvent>();

  // Props
  /** Provide classes to style the stepper header gap. */
  export let gap: CssClasses = 'gap-4';

  // Props (stepper)
  /** Provide the verbiage that represents "Step". */
  export let stepTerm = 'Step';
  /** Provide classes to style the stepper header badges. */
  export let badge: CssClasses = 'variant-filled-surface';
  /** Provide classes to style the stepper header active step badge. */
  export let active: CssClasses = 'variant-filled';
  /** Provide classes to style the stepper header border. */
  export let border: CssClasses = 'border-surface-400-500-token';
  /** Provide the initially selected step*/
  export let start = 0;

  export let currentStep = start;

  // Props (step)
  /** Set the justification for the step navigation buttons. */
  export let justify: CssClasses = 'justify-between';

  // Button (back)
  /** Provide arbitrary classes to style the back button. */
  export let buttonBack: CssClasses = 'variant-ghost';
  /** Set the type of the back button. */
  export let buttonBackType: StepperButton = 'button';
  /** Provide the HTML label content for the back button. */
  export let buttonBackLabel = '&larr; Back';

  // Button (next)
  /** Provide arbitrary classes to style the next button. */
  export let buttonNext: CssClasses = 'variant-filled';
  /** Set the type of the next button. */
  export let buttonNextType: StepperButton = 'button';
  /** Provide the HTML label content for the next button. */
  export let buttonNextLabel = 'Next &rarr;';

  // Button (complete)
  /** Provide arbitrary classes to style the complete button. */
  export let buttonComplete: CssClasses = 'variant-filled-primary';
  /** Set the type of the complete button. */
  export let buttonCompleteType: StepperButton = 'button';
  /** Provide the HTML label content for the complete button. */
  export let buttonCompleteLabel = 'Complete';

  // Props (regions)
  /** Provide arbitrary classes to the stepper header region. */
  export let regionHeader: CssClasses = '';
  /** Provide arbitrary classes to the stepper content region. */
  export let regionContent: CssClasses = '';

  // Classes
  const cBase = 'space-y-4';
  const cHeader = 'flex items-center border-t mt-[15px]';
  const cHeaderStep = '-mt-[15px] transition-all duration-300';
  const cContent = '';

  // Stores
  let state: Writable<StepperState> = writable({ current: start, total: 0 });

  // Event Handlers
  async function onNext(locked: boolean, stepIndex: number) {
    // Allows any forms to submit before the Step is removed from the DOM:
    // https://github.com/skeletonlabs/skeleton/issues/1328
    await new Promise((resolve) => setTimeout(resolve));
    if (locked) return;
    /** @event { step: number, $state: StepperState } next - Fires when the NEXT button is pressed per step.  */
    dispatch('next', { step: stepIndex });
  }
  function onBack(stepIndex: number) {
    /** @event { step: number, $state: StepperState } back - Fires when the BACK button is pressed per step.  */
    dispatch('back', { step: stepIndex });
  }
  function onComplete(stepIndex: number) {
    /** @event { step: number, $state: StepperState } complete - Fires when the COMPLETE button is pressed.  */
    dispatch('complete', { step: stepIndex });
  }

  // Context
  setContext('state', state);
  setContext('stepTerm', stepTerm);
  setContext('gap', gap);
  setContext('justify', justify);
  // ---
  setContext('onNext', onNext);
  setContext('onBack', onBack);
  setContext('onComplete', onComplete);
  // ---
  setContext('buttonBack', buttonBack);
  setContext('buttonBackType', buttonBackType);
  setContext('buttonBackLabel', buttonBackLabel);
  // ---
  setContext('buttonNext', buttonNext);
  setContext('buttonNextType', buttonNextType);
  setContext('buttonNextLabel', buttonNextLabel);
  // ---
  setContext('buttonComplete', buttonComplete);
  setContext('buttonCompleteType', buttonCompleteType);
  setContext('buttonCompleteLabel', buttonCompleteLabel);

  // State
  $: isActive = (step: number) => step === $state.current;
  // Reactive
  $: classesBase = `${cBase} ${$$props.class ?? ''}`;
  $: classesHeader = `${cHeader} ${border} ${gap} ${regionHeader}`;
  $: classesHeaderStep = `${cHeaderStep}`;
  $: classesBadge = (step: number) => (isActive(step) ? active : badge);
  $: classesContent = `${cContent} ${regionContent}`;

  $: $state.current = currentStep;
</script>

<div class="stepper {classesBase}" data-testid="stepper">
  <!-- Header -->
  {#if $state.total}
    <header class="stepper-header {classesHeader}">
      {#each Array.from(Array($state.total).keys()) as step}
        <div class="stepper-header-step {classesHeaderStep}" class:flex-1={isActive(step)}>
          <span class="badge {classesBadge(step)}">
            {isActive(step) ? `${stepTerm} ${step + 1}` : step + 1}
          </span>
        </div>
      {/each}
    </header>
  {/if}
  <!-- Content -->
  <div class="stepper-content {classesContent}">
    <slot />
  </div>
</div>
