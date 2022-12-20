# ACF-pro-shortcode
ACF Field backend code


Get product variations attributes details in shortcode

add_shortcode('spacification-details','ac_product_attributes_callback');
function ac_product_attributes_callback(){
	$html = "";
	global $product;
	$product_attributes = array();
	
	$display_dimensions =  ( $product->has_weight() || $product->has_dimensions() ) ? 1 : 0;

	if ( $display_dimensions && $product->has_weight() ) {
		$product_attributes['weight'] = array(
			'label' => __( 'Weight', 'woocommerce' ),
			'value' => wc_format_weight( $product->get_weight() ),
		);
	}

	if ( $display_dimensions && $product->has_dimensions() ) {
		$product_attributes['dimensions'] = array(
			'label' => __( 'Dimensions', 'woocommerce' ),
			'value' => wc_format_dimensions( $product->get_dimensions( false ) ),
		);
	}
	
	$attributes = $product->get_attributes();
	foreach ( $attributes as $attribute ) {
		$values = array();

		if ( $attribute->is_taxonomy() ) {
			$attribute_taxonomy = $attribute->get_taxonomy_object();
			$attribute_values   = wc_get_product_terms( $product->get_id(), $attribute->get_name(), array( 'fields' => 'all' ) );

			foreach ( $attribute_values as $attribute_value ) {
				$value_name = esc_html( $attribute_value->name );

				if ( $attribute_taxonomy->attribute_public ) {
					$values[] = '<a href="' . esc_url( get_term_link( $attribute_value->term_id, $attribute->get_name() ) ) . '" rel="tag">' . $value_name . '</a>';
				} else {
					$values[] = $value_name;
				}
			}
		} else {
			$values = $attribute->get_options();

			foreach ( $values as &$value ) {
				$value = make_clickable( esc_html( $value ) );
			}
		}

		$product_attributes[ 'attribute_' . sanitize_title_with_dashes( $attribute->get_name() ) ] = array(
			'label' => wc_attribute_label( $attribute->get_name() ),
			'value' => wptexturize( implode( ', ', $values ) ),
		);
	}
	ob_start();
	if( !empty( $product_attributes ) ){
		?><table class="woocommerce-product-attributes shop_attributes"><tbody><?php
		foreach( $product_attributes as $product_attribute_key => $product_attribute ){
			?>
			<tr class="woocommerce-product-attributes-item woocommerce-product-attributes-item--<?php echo $product_attribute_key; ?>">
				<th class="woocommerce-product-attributes-item__label"><?php echo $product_attribute['label']; ?></th>
				<td class="woocommerce-product-attributes-item__value">
					<p><?php echo $product_attribute['value']; ?></p>
				</td>
			</tr>
			<?php
		}
		?></tbody></table><?php
	}
	$html .= ob_get_clean();
	return $html;
}
