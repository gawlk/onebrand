<script>
    import { createEventDispatcher } from 'svelte'
    import Transition from 'svelte-class-transition'

    import Button from './Button.svelte'

    export let value
    export let label
    export let options

    let toggle = false
    let button
    let dropdown
    let positionClasses
    let width

    const dispatch = createEventDispatcher()

    $: if (toggle && width && button) {
        const bb = button.getBoundingClientRect()
        // w-56 = 24 rem = 224px (if base is 16px) / 2 = 112
        const left = bb.left - 112
        const right = bb.right + 112
        console.log(left, right, width);
        if (left >= 0 && right > width - 1) {
            positionClasses = 'origin-top-right right-2'
        } else if (left < 0 && right < width) {
            positionClasses = 'origin-top-left left-2'
        } else {
            positionClasses = 'origin-center -right-15'
        }
    }

    const onClickOutside = (event) => {
        if (toggle) {
            const x = event.clientX
            const y = event.clientY

            const bb = button.getBoundingClientRect()
            const bd = dropdown.getBoundingClientRect()

            const between = (low, value, high) => value >= low && value <= high

            if (
                ! (between(bb.left, x, bb.right) && between(bb.top, y, bb.bottom))
                && ! (between(bd.left, x, bd.right) && between(bd.top, y, bd.bottom))
            ) {
                toggle = false
            }
        }
    }
</script>

<svelte:window bind:innerWidth={width} on:click={onClickOutside} />

<div class="
    px-2
    py-0.5
    relative
    inline-block
">
    <div bind:this={button}>
        <Button
            type="button"
            on:click={() => { toggle = ! toggle }}
            {label}
        >
            {value}
            <svg class="-mr-1 ml-2 h-6 w-6" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
            </svg>
        </Button>
    </div>

    <Transition
        {toggle}
        inState="opacity-0 scale-95"
        onState="opacity-100 scale-100"
        transitions="transition transform"
        inTransition="ease-out duration-100"
        outTransition="ease-in duration-75"
    >
        <div
            bind:this={dropdown}
            class="
            absolute
            z-20
            mt-1
            w-56
            rounded-lg
            shadow-lg
            border-4
            border-black
            bg-white
            p-2
            {positionClasses}
        ">
            {#each options as option}
                <button
                    on:click={() => { dispatch('select', { value: option }); toggle = false }}
                    {label}
                    class="
                    w-full
                    text-left
                    block
                    text-xl
                    font-extrabold
                    rounded-md
                    px-2
                    py-0.5
                    hover:bg-gray-100
                ">
                    {option}
                </button>
            {/each}
        </div>
    </Transition>
</div>
