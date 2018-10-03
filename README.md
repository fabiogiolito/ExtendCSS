# ExtendCSS

An _experimental_ Utility CSS framework that pushes the limits of SASS to write CSS classes **programatically**.

> Still in development

Inspired by [Tailwind CSS](https://tailwindcss.com), but made purely with SCSS, without Javascript or any other language.


## What is a Utility CSS Framework?

It is a CSS Framework for rapid prototyping and layout. It is focused on providing you with utility classes that define specific CSS properties,  like `.bg-blue` or `.text-center`.

This approach differs from frameworks [Bootstrap](http://getbootstrap.com) that give you opinionated component classes that you then have to customize or apply themes to make them your own.

Utility classes give you more freedom to create and customize your UI any way you like while still helping you follow some structure:

```html
<button class="bg-blue text-white p-y-2 p-x-4 rounded hover:bg-blue-darker">
    Click me!
</button>
```


## @Extend mode

ExtendCSS also generates placeholder `%` classes that you can `@extend` to create your own components. So if all your buttons have the same classes you should consider them to a new style:

```scss
.btn {
    @extend %bg-blue, %text-white, %p-y-2, %p-x-4, %rounded, %hover\:bg-blue-darker;
}
```
```html
<button class="btn">Click me!</button>
```

> If you prefer this component approach and all you need are the placeholder `%` classes you can disable the regular `.` classes by setting the `$generateClasses` varialble to `false` on the main `scss/_application.scss`.


## Generating classes programatically

To create classes programaticallty we use the `make()` mixin. It takes four parameters:

```scss
make($base, $properties, $values, $variants);
```

1. **$base**: The base name for the class.

    ```scss
    $base: 'border';

    // .#{$base}-2 { … }

    .border-2 { border-width: 2px; }
    ```

2. **$properties**: The css property you're modifying with the class. This can be a string for something straightforward like `background-color` or a key/value list generated multiple modifiers classes:

    ```scss
    $properties: (
        't': 'border-top-weight', 
        'b': 'border-bottom-weight',
        …
    );

    // .border-#{$key}-2 { #{$value}: 2px; }

    .border-t-2 { border-top-weight: 2px; }
    .border-b-2 { border-bottom-weight: 2px; }
    ```

    The property value can also be a list for multiple properties:
    ```scss
    $properties: (
        'y': ('border-top-width', 'border-bottom-width'),
        …
    );

    // .border-#{$propertyKey}-2 { #{$propertyValue}: 2px; }

    .border-y-2 { 
        border-top-width: 2px; 
        border-bottom-width: 2px; 
    }
    ```

    If the key is `'default'` it won't be used in the class name:

    ```scss
    $properties: (
        'default': 'border-width',
        't': 'border-top-weight',
        …
    );

    .border-2 { border-width: 2px; }
    .border-t-2 { border-top-width: 2px; }
    ```

3. **$values**: A key/value list with the class modifier and the property value.
    
    ```scss
    $value: (
        '2': '2px',
        …
    );

    // .border-#{$key} { border-width: #{$value}; }

    .border-2 { border-width: 2px; }
    ```

4. **$variants**: Variants are special prefixes to your classes. There are three types of variants:

    1. **Special selector variants**: `hover`, `active`, `first-child`…
    
    ```scss
    $variants: ('hover', 'active', …);

    // .#{$variant}:bg-blue

    .bg-blue { background-color: #00F; }
    .hover\:bg-blue {
        &:hover { background-color: #00F; }
    }
    ```

    ```html
    <button class="bg-blue hover:bg-red active:bg-green">
        I'm red on hover, and green on click
    </button>
    ```

    2. **Screen size variants**: `xs`, `md`, `lg`…

    ```scss
    $responsive: (
        'sm': '(min-width: 576px)',
        'md': '(min-width: 768px)',
        …
    );
    $variants: ('hover', $responsive);

    // .#{$responsiveKey}:bg-blue

    @media (min-width: 768px) {
        .xs\:bg-blue { background-color: #00F; }
    }
    ```

    ```html
    <button class="bg-blue md:bg-red lg:bg-green">
        I'm blue on phones, red on tablets, green on desktop
    </button>
    ```

    3. **Parent target variants**: `group-hover`

    Sometimes you need to change the style of an element when you hover their parent. This is where group-hover is helpful:

    ```scss
    $variants: ('group:hover .group-hover', …);

    // .#{variant}:bg-blue

    .group:hover .group-hover\:bg-blue { … }
    ```
    
    ```html
    <div class="group border p-4 hover:bg-blue">
        <p class="group-hover:text-white">Some text</p>
        <button class="bg-blue text-white group-hover:bg-white group-hover:text-blue">
            Click me
        </button>
    </div>
    ```

    You can use this pattern in creative ways to create other parent variants too:

    ```scss
    $variants: ('.dropdown:focus .dropdown-open', …);
    ```
    ```html
    <div class="dropdown outline-none" tabindex="0">
        <button class="p-2">Options ▾</button>
        <div class="p-4 border hidden absolute dropdown-open:block">
            Dropdown menu…
        </div>
    </div>
    ```


## Bringing it all together

The main SASS file (`scss/_application.scss`) imports the functions and different config files we're going to need.

Then it defines a list of `$classes` we want to loop through and run the `make()` mixin.

```scss
// ($base, $properties, $values, $variants),
$classes: (
    ('border', $borderWidthProperties, $borderWidths, $borderWidthVariants),
    ('border', 'border-color', $borderColors, $borderColorVariants),
    ('border', 'border-style', $borderStyles, $borderStyleVariants),
    ('rounded', $borderRadiusProperties, $borderRadius, $borderRadiusVariants),
    …
);
```


## Customization

Take this line as an example:
```scss
('border', 'border-color', $borderColors, $borderColorVariants),
```
The variables `$borderColors` and `$borderColorVariants` are defined in the `sass/config/borders.scss` file, and you can customize them there if needed.

By default `$borderColors` is equal to the main list of `$colors`. If you don't need colorful borders in your UI you can change that variable to only the border colors you need, for example changing it to `('faded': 'rgba(0,0,0,0.1)')` will generate a single `.border-faded` class.

Likewise `$borderVariants` by default will generates lots of variants, if you don't plan on changing border colors on hover, or depending on screen size, then you can edit that variable to remove those variants. If you need no variants at all you can set it equal to an empty list `()`.
