<?php
/*
Plugin Name:       Random Featured Image
Plugin URI:        https://example.com/plugins/random-featured-image
Description:       При первой публикации поста автоматически назначает случайное изображение из медиабиблиотеки как featured image.
Version:           1.0.0
Author:            Your Name
Author URI:        https://example.com
Text Domain:       random-featured-image
Domain Path:       /languages
License:           GPLv2 or later
*/

if ( ! defined( 'ABSPATH' ) ) {
    exit; // Защита от прямого выполнения
}

if ( ! class_exists( 'RFI_Random_Featured_Image' ) ) {

    class RFI_Random_Featured_Image {

        public function __construct() {
            // Загрузка переводов (при наличии .mo/.po)
            add_action( 'plugins_loaded', [ $this, 'load_textdomain' ] );
            // Хук на сохранение записи
            add_action( 'save_post', [ $this, 'maybe_set_random_image' ], 10, 3 );
        }

        /**
         * Загружает .mo/.po для локализации
         */
        public function load_textdomain() {
            load_plugin_textdomain(
                'random-featured-image',
                false,
                dirname( plugin_basename( __FILE__ ) ) . '/languages'
            );
        }

        /**
         * При первом сохранении опубликованного поста устанавливает случайное featured image
         *
         * @param int     $post_id ID сохраняемого поста.
         * @param WP_Post $post    Объект поста.
         * @param bool    $update  true — это обновление уже существующего поста.
         */
        public function maybe_set_random_image( $post_id, $post, $update ) {
            // Игнорируем ревизии
            if ( wp_is_post_revision( $post_id ) ) {
                return;
            }

            // Только для типа post и только при первом создании публикации
            if ( 'post' !== get_post_type( $post ) || $update ) {
                return;
            }

            // Проверяем статус — должен быть именно publish
            if ( 'publish' !== $post->post_status ) {
                return;
            }

            // Пользователь должен иметь право редактировать этот пост
            if ( ! current_user_can( 'edit_post', $post_id ) ) {
                return;
            }

            // Если миниатюра уже установлена — уходим
            if ( has_post_thumbnail( $post_id ) ) {
                return;
            }

            // Получаем ID всех картинок из медиабиблиотеки
            $images = get_posts( [
                'post_type'      => 'attachment',
                'post_mime_type' => 'image',
                'numberposts'    => -1,
                'post_status'    => 'inherit',
                'fields'         => 'ids',
            ] );

            if ( empty( $images ) ) {
                return;
            }

            // Выбираем случайный ID
            $random_id = $images[ array_rand( $images ) ];

            // Пытаемся установить; в случае ошибки пишем в лог
            if ( ! set_post_thumbnail( $post_id, $random_id ) ) {
                error_log( sprintf(
                    'RFI: не удалось установить featured image для поста %d (image ID %d)',
                    $post_id,
                    $random_id
                ) );
            }
        }
    }

    new RFI_Random_Featured_Image();
}
