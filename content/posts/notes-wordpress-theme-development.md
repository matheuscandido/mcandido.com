+++
author = "Matheus Candido"
title = "Notes on WordPress Theme and Plugin Development"
date = "2020-11-19"
description = "I'll try to write down all I know about WP theme development."
tags = [
    "wordpress",
]
+++

These are notes for future self-reference on WordPress development. I'm publishing because they can be useful for concentrating all the information about many topics in a single place.

## Widgets

Widgets are very useful for providing a drag-and-drop user interface for simillar self-contained components. 

First register the new sidebar on `functions.php`:
```
function register_ebooks_sidebar() {
	register_sidebar(array(
		'name'          => 'eBooks Sidebar',
		'id'            => 'ebooks_sidebar',
	));
}
add_action( 'widgets_init', 'register_ebooks_sidebar' );
```

Then place the sidebar on its place in any page wanted (`page-something.php`):
```
<?php if ( is_active_sidebar( 'ebooks_sidebar' ) ) : ?>
    <?php dynamic_sidebar( 'ebooks_sidebar' ); ?>
<?php endif; ?>
```

Create the custom widget defining a class inherited from `WP_Widget`. (It needs to be required or declared on `functions.php` to work):
```
class Ebook_Widget extends WP_Widget {

	/**
	 * Sets up the widgets name, description and other properties
	 */
	public function __construct() {

        // adds script for native wordpress media uploader
        add_action('admin_enqueue_scripts', array($this, 'scripts'));

		$widget_ops = array( 
			'classname' => 'ebook_widget',
			'description' => 'Some description here',
		);
		parent::__construct( 'ebook_widget', 'Ebook', $widget_ops );
    }
    
    // Adds more scripts for enabling WP media uploader
    public function scripts()
    {
        wp_enqueue_script( 'media-upload' );
        wp_enqueue_media();
        wp_enqueue_script('widget-ebook', get_template_directory_uri() . '/js/ebook-widget.js', array('jquery'));
    }

	/**
	 * Outputs the content of the widget
	 *
	 * @param array $args
	 * @param array $instance
	 */
    public function widget( $args, $instance ) { ?>

<div class="ebook">
    <div class="ebook__image-container">
        <img src="<?php echo esc_url($instance['image_url']) ?>" alt="">
        <div class="ebook__image-container__overlay"></div>
    </div>
    <div class="ebook__info">
        <span class="ebook__info__title"><?php echo $instance['title'] ?></span>
        <p><?php echo $instance['description']; ?>
        </p>
        <div class="ebook__info__price">
            <div>
            </div>
            <?php if(strlen($instance['enabled']) > 0) : ?>
            <a id="ebooks__card__action" href="<?php echo $instance['link'] ?>"
                class="button button--regular button--transparent-pink"><?php echo $instance['button_text'] ?></a>
            <?php else : ?>
            <a id="ebooks__card__action" href="<?php echo $instance['link'] ?>"
                class="button button--regular button--transparent-disabled"><?php echo $instance['button_text'] ?></a>
            <?php endif; ?>
        </div>
    </div>
</div>
<?php }

	/**
	 * Outputs the options form on admin page
	 *
	 * @param array $instance The widget options
	 */
	public function form( $instance ) {
 
        $title = ! empty( $instance['title'] ) ? $instance['title'] : esc_html__( '', 'text_domain' );
        $description = ! empty( $instance['description'] ) ? $instance['description'] : esc_html__( '', 'text_domain' );
        $link = ! empty( $instance['link'] ) ? $instance['link'] : esc_html__( '', 'text_domain' );
        $button_text = ! empty( $instance['button_text'] ) ? $instance['button_text'] : esc_html__( '', 'text_domain' );
        $image_url = ! empty( $instance['image_url'] ) ? $instance['image_url'] : esc_html__( '', 'text_domain' );
        $enabled = ! empty( $instance['enabled'] ) ? $instance['enabled'] : esc_html__( '', 'text_domain' );
        ?>
<p>
    <label
        for="<?php echo esc_attr( $this->get_field_id( 'title' ) ); ?>"><?php echo esc_html__( 'Título:', 'text_domain' ); ?></label>
    <input class="widefat" id="<?php echo esc_attr( $this->get_field_id( 'title' ) ); ?>"
        name="<?php echo esc_attr( $this->get_field_name( 'title' ) ); ?>" type="text"
        value="<?php echo esc_attr( $title ); ?>">
</p>
<p>
    <label
        for="<?php echo esc_attr( $this->get_field_id( 'description' ) ); ?>"><?php echo esc_html__( 'Descrição:', 'text_domain' ); ?></label>
    <textarea class="widefat" id="<?php echo esc_attr( $this->get_field_id( 'description' ) ); ?>"
        name="<?php echo esc_attr( $this->get_field_name( 'description' ) ); ?>" type="text" cols="30"
        rows="10"><?php echo esc_attr( $description ); ?></textarea>
</p>
<p>
    <label
        for="<?php echo esc_attr( $this->get_field_id( 'link' ) ); ?>"><?php echo esc_html__( 'Link do Botão:', 'text_domain' ); ?></label>
    <input class="widefat" id="<?php echo esc_attr( $this->get_field_id( 'link' ) ); ?>"
        name="<?php echo esc_attr( $this->get_field_name( 'link' ) ); ?>" type="text"
        value="<?php echo esc_attr( $link ); ?>">
</p>
<p>
    <label
        for="<?php echo esc_attr( $this->get_field_id( 'button_text' ) ); ?>"><?php echo esc_html__( 'Texto do Botão:', 'text_domain' ); ?></label>
    <input class="widefat" id="<?php echo esc_attr( $this->get_field_id( 'button_text' ) ); ?>"
        name="<?php echo esc_attr( $this->get_field_name( 'button_text' ) ); ?>" type="text"
        value="<?php echo esc_attr( $button_text ); ?>">
</p>
<p>
    <label for="<?php echo $this->get_field_id( 'enabled' ); ?>">Ativo</label>
    <input class="checkbox" type="checkbox" <?php checked( $instance[ 'enabled' ], 'on' ); ?>
        id="<?php echo $this->get_field_id( 'enabled' ); ?>" name="<?php echo $this->get_field_name( 'enabled' ); ?>" />

</p>
<p>
    <label
        for="<?php echo esc_attr( $this->get_field_id( 'image_url' ) ); ?>"><?php echo esc_html__( 'Imagem:', 'text_domain' ); ?></label>
    <input class="widefat" id="<?php echo $this->get_field_id( 'image_url' ); ?>"
        name="<?php echo $this->get_field_name( 'image_url' ); ?>" type="text"
        value="<?php echo esc_url( $image_url ); ?>" />
    <button class="upload_image_button button button-primary">Upload de Imagem</button>
</p>
<?php
 
    }

	/**
	 * Processing widget options on save
	 *
	 * @param array $new_instance The new options
	 * @param array $old_instance The previous options
	 *
	 * @return array
	 */
	public function update( $new_instance, $old_instance ) {
		$instance = array();
 
        $instance['title'] = ( !empty( $new_instance['title'] ) ) ? strip_tags( $new_instance['title'] ) : '';
        $instance['description'] = ( !empty( $new_instance['description'] ) ) ? $new_instance['description'] : '';
        $instance['link'] = ( !empty( $new_instance['link'] ) ) ? $new_instance['link'] : '';
        $instance['button_text'] = ( !empty( $new_instance['button_text'] ) ) ? $new_instance['button_text'] : '';
        $instance['image_url'] = ( !empty( $new_instance['image_url'] ) ) ? $new_instance['image_url'] : '';
        $instance['enabled'] = ( !empty( $new_instance['enabled'] ) ) ? $new_instance['enabled'] : '';
 
        return $instance;
	}
}

```

## Customizer Fields

WordPress themes can have parameters to be edited on the Customize page. All its set up can be done on `functions.php` or requiring a custom PHP file there.

First, register the customization fields on the appropriate hook:
```
function my_theme_customizer($wp_customize){
    // All the sections, settings and inputs will be called here.
}
add_action('customzie_register', my_theme_customizer);
```

All the code bellow will be added to this customizer function declared.

Add a section:
```
$wp_customize->add_section( 'section_name' , array(
    'title'      => __( 'Visible Section Name', 'mytheme' ),
    'priority'   => 30,
) );
```

Add a setting:
```
$wp_customize->add_setting( 'header_textcolor' , array(
    'default'   => '#000000',
    'transport' => 'refresh',
) );
```

And then, add a control to change the setting
```
$wp_customize->add_control( new WP_Customize_Color_Control( $wp_customize, 'header_textcolor', array(
	'label'      => __( 'Header Color', 'mytheme' ),
	'section'    => 'section_name',
	'settings'   => 'header_textcolor',
) ) );
```
It is useful to check the documentation of any classes instantiated for custom attributes to be tweaked and used.

## Custom Post Types

## Custom Meta Fields and Metaboxes

## Categories/Taxonomies

## Custom Pages

## Custom Forms

## WooCommerce Theme Development

## WooCommerce Plugin Development