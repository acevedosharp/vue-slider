<template>
  <div ref="sliderContainer"
       class="slider-container"
       @mouseenter="deactivateAutoMoveWithChecks"
       @mouseover="deactivateAutoMoveWithChecks"
       @mouseleave="activateAutoMoveWithChecks">

    <button ref="leftButton"
            class="inline side-button"
            @click="moveSlider('left')">
        <span :class="this.leftButtonDisabled ? 'disabled-arrow' : 'normal-arrow'">
          <
        </span>
    </button>

    <div ref="slider"
         class="slider inline disabled-touch-scrolling">
      <slot/>
    </div>

    <button ref="rightButton"
            class="inline side-button"
            @click="moveSlider('right')">
        <span :class="this.rightButtonDisabled ? 'disabled-arrow' : 'normal-arrow'">
          >
        </span>
    </button>
  </div>
</template>

<script>
export default {
  name: "Slider",
  data() {
    return {
      leftButtonDisabled: false,
      rightButtonDisabled: false,

      elementWidth: 0,
      distancePerStep: 0,
      elementLeftMargin: 0,
      elementRightMargin: 0,
      autoMoveIntervalId: [],
      isFirst: true,
      mouseDown: false,

      // custom sliding animation
      isInMovement: false,
      targetSlider: {},
      interpolation: [],
      callback: () => {
      },
      currentIndex: 0,

      dragStep: {
        currDiff: 0,
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
    },
    width: {
      type: Number,
      required: true
    },
    sliderHeight: {
      type: String,
      required: true
    },
    interpolationType: {
      type: String,
      required: false,
      default: 'smooth'
    },
    infiniteLooping: {
      type: Boolean,
      required: false,
      default: true
    }
  },
  mounted() {
    this.updateElementWidth()
    this.updateSideButtons()

    this.$refs.slider.style.height = this.sliderHeight

    if (this.doAutoMove) {
      this.autoMoveIntervalId.push(setInterval(() => this.doAutoMoveWithChecks(), this.autoMovePeriod))
    }

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

    this.setupEvents()
  },
  methods: {
    // Events
    handleDragStart(e) {
      this.dragStep.initialX = e.screenX
      this.dragStep.initialScrollLeft = this.$refs.slider.scrollLeft
    },
    handleTouchStart(e) {
      this.deactivateAutoMoveWithChecks()
      if (!this.isInMovement) {
        this.dragStep.initialX = e.touches[0].clientX
        this.dragStep.initialScrollLeft = this.$refs.slider.scrollLeft
      }
    },
    handleTouchMove(e) {
      if (!this.isInMovement) {
        const difference = this.dragStep.initialX - e.touches[0].clientX
        this.currDiff = difference
        this.dragStep.doMovement = Math.abs(difference) > this.distancePerStep * this.boundaryCoefficient;
        this.dragStep.targetDirection = Math.sign(difference) === -1 ? 'left' : 'right'
        this.$refs.slider.scrollLeft = this.dragStep.initialScrollLeft + difference * 0.7
      }
    },
    handleDragOver(e) {
      const difference = this.dragStep.initialX - e.screenX
      this.currDiff = difference
      this.dragStep.doMovement = Math.abs(difference) > this.distancePerStep * this.boundaryCoefficient;
      this.dragStep.targetDirection = Math.sign(difference) === -1 ? 'left' : 'right'
      this.$refs.slider.scrollLeft = this.dragStep.initialScrollLeft + difference * 0.7
    },
    handleDragEnd(e) {
      this.deactivateAutoMoveWithChecks()
      this.activateAutoMoveWithChecks()
      if (this.currDiff === 0) return

      if (this.dragStep.doMovement) {
        this.moveSlider(this.dragStep.targetDirection)
      } else {
        this.doScrollInitiator(this.dragStep.initialScrollLeft, () => {
        })
      }
      this.currDiff = 0
    },
    handleMouseMove(e) {
      if (this.mouseDown) {
        this.handleDragOver(e)
      }
    },
    handleMouseLeave(e) {
      this.mouseDown = false
    },
    handleMouseDown(e) {
      if (!this.isInMovement) {
        this.handleDragStart(e)
        this.mouseDown = true
      }
    },
    handleMouseUp(e) {
      if (this.mouseDown) {
        this.handleDragEnd(e)
        this.mouseDown = false
      }
    },
    setupEvents() {
      const slider = this.$refs.slider
      slider.addEventListener("touchstart", this.handleTouchStart, false);
      slider.addEventListener("touchmove", this.handleTouchMove, false);
      slider.addEventListener("touchend", this.handleDragEnd, false);
      slider.addEventListener("touchcancel", this.handleDragEnd, false);
      slider.addEventListener('mousemove', this.handleMouseMove, false)
      slider.addEventListener('mouseleave', this.handleMouseLeave, false)
      slider.addEventListener('mousedown', this.handleMouseDown, false)
      slider.addEventListener('mouseup', this.handleMouseUp, false)
    },


    // automatic movement
    activateAutoMoveWithChecks() {
      this.mouseOver = false
      if (this.doAutoMove)
        this.autoMoveIntervalId.push(setInterval(() => this.doAutoMoveWithChecks(), this.autoMovePeriod))
    },
    deactivateAutoMoveWithChecks() {
      this.mouseOver = true
      if (this.doAutoMove) {
        this.autoMoveIntervalId.forEach((id) => clearInterval(id))
        this.autoMoveIntervalId = []
      }
    },
    doAutoMoveWithChecks() {
      this.moveSlider('right')
    },


    // helpers
    updateSideButtons() {
      const slider = this.$refs.slider
      let leftButtonStatus
      let rightButtonStatus
      if (slider.childNodes.length === 0) {
        leftButtonStatus = true
        rightButtonStatus = true
      } else if (slider.childNodes.length * this.distancePerStep < slider.scrollWidth) {
        leftButtonStatus = true
        rightButtonStatus = true
      } else if (slider.scrollLeft === 0) {
        leftButtonStatus = true
        rightButtonStatus = false
      } else if (!this.infiniteLooping && this.isSliderShowingLastElement()) {
        leftButtonStatus = false
        rightButtonStatus = true
      } else {
        leftButtonStatus = false
        rightButtonStatus = false
      }
      this.$refs.leftButton.disabled = leftButtonStatus
      this.$refs.rightButton.disabled = rightButtonStatus

      this.leftButtonDisabled = leftButtonStatus
      this.rightButtonDisabled = rightButtonStatus
    },
    isSliderShowingLastElement() {
      const slider = this.$refs.slider
      return (slider.scrollLeft + slider.offsetWidth) === slider.scrollWidth
    },
    updateElementWidth() {
      const childNodes = this.$refs.slider.childNodes
      if (childNodes.length > 0) {
        const singleElement = childNodes[0]
        const singleElementWidth = this.width
        const computedStyle = window.getComputedStyle(singleElement)
        this.elementLeftMargin = parseInt(computedStyle.marginLeft)
        this.elementRightMargin = parseInt(computedStyle.marginRight)
        this.elementWidth = singleElementWidth
        this.distancePerStep = singleElementWidth + this.elementLeftMargin + this.elementRightMargin
      } else {
        this.elementWidth = 0
        this.distancePerStep = 0
      }
    },


    // slider scrolling
    doScrollInitiator(targetX, callback) {
      if (!this.isInMovement) {
        const slider = this.$refs.slider
        const max = Math.max(targetX, slider.scrollLeft)
        const min = Math.min(targetX, slider.scrollLeft)
        const fullRangeBetween = [...Array(Math.round(max - min + 1)).keys()].map(i => i + min)

        if (targetX < slider.scrollLeft)
          fullRangeBetween.reverse()

        let interpolation = []
        if (this.interpolationType === 'linear') {
          const stepSize = (max - min) / 16
          interpolation.push(fullRangeBetween[0])
          for (let i = 1; i <= 14; i++) {
            interpolation.push(fullRangeBetween[Math.round(stepSize * i)])
          }
          interpolation.push(fullRangeBetween.lastItem)
        } else if (this.interpolationType === 'smooth') {
          const mask = [1, 1.1, 1.2, 1.35, 1.5, 1.65, 1.8, 2, 2, 1.8, 1.65, 1.5, 1.35, 1.2, 1.1, 1]
          let maskAcc = []
          let weightedSum = -1
          for (let i = 0; i < mask.length; i++) {
            weightedSum += mask[i]
            maskAcc.push(weightedSum)
          }

          // normalize accumulated mask
          maskAcc = maskAcc.map(n => n / weightedSum)

          const frb = fullRangeBetween.length - 1
          interpolation.push(fullRangeBetween[0])
          for (let i = 1; i <= mask.length - 2; i++) {
            interpolation.push(fullRangeBetween[Math.round(maskAcc[i] * frb)])
          }
          interpolation.push(fullRangeBetween[fullRangeBetween.length - 1])
        } else {
          throw Error('No such interpolationType')
        }
        this.targetSlider = slider
        this.interpolation = interpolation
        this.callback = callback
        this.currentIndex = 0

        this.isInMovement = true
        this.doScrollRecursive()
      }
    },
    doScrollRecursive() {
      if (this.currentIndex < 16) {
        this.targetSlider.scrollLeft = this.interpolation[this.currentIndex]
        this.currentIndex += 1
        setTimeout(() => this.doScrollRecursive(), 16.67)
      } else {
        this.callback()
        this.isInMovement = false
      }
    },
    moveSlider(direction) {
      if (this.isInMovement)
        return

      if (!this.infiniteLooping && this.isSliderShowingLastElement() && direction === 'right')
        return

      const slider = this.$refs.slider
      let remainder = slider.scrollLeft % this.distancePerStep
      let distanceToMoveBy
      if (remainder === 0) {
        distanceToMoveBy = this.distancePerStep * this.step
      } else {
        const offset = direction === 'right' ? this.distancePerStep - remainder : remainder
        distanceToMoveBy = offset + (this.step - 1) * this.distancePerStep
      }
      if (direction === 'left') distanceToMoveBy *= -1
      let targetScrollX = slider.scrollLeft + distanceToMoveBy

      if (this.isSliderShowingLastElement() && direction === 'right')
        targetScrollX = 0

      this.deactivateAutoMoveWithChecks()
      this.doScrollInitiator(targetScrollX, () => {
        this.updateSideButtons()
        this.activateAutoMoveWithChecks()
      })
    },
  },

  destroyed() {
    this.deactivateAutoMoveWithChecks()
    window.onresize = null
  }
}
</script>

<style scoped>
.slider-container {
  width: 100%
}

::-webkit-scrollbar {
  display: none;
}

.disabled-touch-scrolling {
  touch-action: pan-y;
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
  width: 100%
}

.side-button {
  width: 100px;
  font-size: 50px;
  color: rgb(0, 106, 255);
}

.normal-arrow {
  font-size: 35px;
  color: rgb(0, 106, 255);
}

.disabled-arrow {
  font-size: 35px;
  color: rgba(0, 106, 255, 0.3);
}
</style>