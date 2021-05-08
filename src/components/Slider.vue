<template>
  <div>
    <div ref="sliderContainer"
         class="slider-container"
         @mouseenter="deactivateAutoMove"
         @mouseleave="activateAutoMove">

      <button ref="leftButton"
              class="inline"
              @click="moveSlider('left')">
        <
      </button>

      <div ref="slider"
           class="slider inline disabled-touch-scrolling"
           @dragstart="handleDragStart"
           @dragover="handleDragOver"
           @dragend="handleDragEnd"
           draggable="true">
        <slot/>
      </div>

      <button ref="rightButton"
              class="inline"
              @click="moveSlider('right')">
        >
      </button>

    </div>
  </div>
</template>

<script>
export default {
  name: "Slider",
  data() {
    return {
      elementWidth: 0,
      distancePerStep: 0,
      elementLeftMargin: 0,
      elementRightMargin: 0,
      autoMoveIntervalId: 0,
      dragStep: {
        initialScrollLeft: 0,
        initialX: 0,
        targetDirection: 'right',
        doMovement: false
      }
    }
  },
  props: {
    step: {
      type: Number,
      required: true
    },
    boundaryCoefficient: {
      type: Number,
      required: false,
      default: 0.25
    },
    doAutoMove: {
      type: Boolean,
      required: true
    },
    autoMovePeriod: {
      type: Number,
      required: false,
      default: 2000
    },
    hideButtonsBoundary: {
      type: Number,
      required: false,
      default: 960
    }
  },
  mounted() {
    this.updateElementWidth()

    if (this.doAutoMove) {
      this.autoMoveIntervalId = setInterval(() => this.doAutoMoveWithChecks(), this.autoMovePeriod)
    }

    // drag events for mobile
    const slider = this.$refs.slider
    slider.addEventListener("touchstart", this.handleTouchStart, false);
    slider.addEventListener("touchmove", this.handleTouchMove, false);
    slider.addEventListener("touchend", this.handleDragEnd, false);

    const setButtonsDisplay = (label) => {
      this.$refs.leftButton.style.display = label
      this.$refs.rightButton.style.display = label
    }

    if (window.innerWidth <= this.hideButtonsBoundary) {
      setButtonsDisplay('none')
    }

    window.onresize = (e) => {
      if (e.target.innerWidth <= this.hideButtonsBoundary) {
        setButtonsDisplay('none')
      } else {
        setButtonsDisplay('inline-block')
      }
    }
  },
  methods: {
    activateAutoMove() {
      this.autoMoveIntervalId = setInterval(() => this.doAutoMoveWithChecks(), this.autoMovePeriod)
    },
    deactivateAutoMove() {
      clearInterval(this.autoMoveIntervalId)
    },
    doAutoMoveWithChecks() {
      this.moveSlider('right')
    },
    handleDragStart(e) {
      this.dragStep.initialX = e.screenX
      this.dragStep.initialScrollLeft = this.$refs.slider.scrollLeft
      // do not show ghost image
      const img = new Image();
      img.src = 'data:image/gif;base64,R0lGODlhAQABAIAAAAUEBAAAACwAAAAAAQABAAACAkQBADs=';
      e.dataTransfer.setDragImage(img, 0, 0);
    },
    handleTouchStart(e) {
      this.dragStep.initialX = e.touches[0].clientX
      this.dragStep.initialScrollLeft = this.$refs.slider.scrollLeft
    },
    handleDragOver(e) {
      const difference = this.dragStep.initialX - e.screenX

      this.dragStep.doMovement = Math.abs(difference) > this.distancePerStep * 0.25;

      this.dragStep.targetDirection = Math.sign(difference) === -1 ? 'left' : 'right'

      this.$refs.slider.scrollLeft = this.dragStep.initialScrollLeft + difference * 0.7
    },
    handleTouchMove(e) {
      const difference = this.dragStep.initialX - e.touches[0].clientX

      this.dragStep.doMovement = Math.abs(difference) > this.distancePerStep * 0.25;

      this.dragStep.targetDirection = Math.sign(difference) === -1 ? 'left' : 'right'

      this.$refs.slider.scrollLeft = this.dragStep.initialScrollLeft + difference * 0.7
    },
    handleDragEnd(e) {
      if (this.dragStep.doMovement) {
        this.moveSlider(this.dragStep.targetDirection)
      } else {
        this.$refs.slider.scroll({
          top: 0,
          left: this.dragStep.initialScrollLeft,
          behavior: 'smooth'
        })
      }
    },
    moveSlider(direction) {
      const slider = this.$refs.slider
      if ((slider.scrollLeft + slider.offsetWidth) === slider.scrollWidth) {
        this.deactivateAutoMove()
        slider.scroll({
          top: 0,
          left: 0,
          behavior: 'smooth'
        })
        this.activateAutoMove()
        return
      }

      let remainder = slider.scrollLeft % this.distancePerStep
      let distanceToMoveBy

      if (remainder === 0) {
        distanceToMoveBy = this.distancePerStep * this.step
      } else {
        const offset = direction === 'right' ? this.distancePerStep - remainder : remainder
        distanceToMoveBy = offset + (this.step - 1) * this.distancePerStep
      }

      if (direction === 'left') distanceToMoveBy *= -1

      const targetScrollX = slider.scrollLeft + distanceToMoveBy

      slider.scroll({
        top: 0,
        left: targetScrollX,
        behavior: 'smooth'
      })
    },
    updateElementWidth() {
      const childNodes = this.$refs.slider.childNodes
      if (childNodes.length > 0) {
        const singleElement = childNodes[0]
        const singleElementWidth = singleElement.getBoundingClientRect().width
        const computedStyle = window.getComputedStyle(singleElement)

        this.elementLeftMargin = parseInt(computedStyle.marginLeft)
        this.elementRightMargin = parseInt(computedStyle.marginRight)

        this.elementWidth = singleElementWidth
        this.distancePerStep = singleElementWidth + this.elementLeftMargin + this.elementRightMargin
      } else {
        this.elementWidth = 0
        this.distancePerStep = 0
      }
    }
  }
}
</script>

<style scoped>
::-webkit-scrollbar {
  display: none;
}

.disabled-touch-scrolling {
  touch-action: none
}

.inline {
  display: inline-block;
}

.slider-container {
  display: flex;
}

.slider {
  white-space: nowrap;
  overflow-x: scroll;
  overflow-scrolling: touch;
}
</style>