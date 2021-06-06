# vue-slider
To use copy the component from Slider.vue and use in your project.

Props:
- **step:** Number of elements every slide movement moves by.
- **boundaryCoefficient:** When tapping, how much horizontal difference must be between the tap starting position and the tap end to do a slide movement (single child element width * coefficient). Between 0 and 1.
- **doAutoMove:** Whether the slider does slide movements by itself or not.
- **autoMovePeriod:** How often (in milliseconds) does the slider do slide movements.
- **hideButtonsBoundary:** Viewport width at which the side navigation buttons disappear.
- **width:** Single element horizontal space (width + margin (less than the full margin; if equal margin in both sides it could be width + margin/2 but this prop should be tunned)).
- **interpolationType:** Interpolation type to use in the custom animation. Options: {'linear', 'smooth'}
- **infiniteLooping:** Whether when the slider reaches the end it should go back to the first element or stop there.
- **sliderHeight:** The height the slider div has, it is required and tunned once. Keep in mind the height of the scrollbar in Firefox which is not hidden and transforms on child elements when being hovered.

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
