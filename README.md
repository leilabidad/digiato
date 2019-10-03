<?php
/*
/*
Plugin Name: بنر برای دیجیاتو
Description: پلاگینی جهت تست و برای ساخت بنر در سایت دیجیاتو
Version:     1.0
Author:      لیلا بیداد
Author URI:  leilabidad.com
License:     GPL2

/**
 * PART 1. Defining Custom Database Table
 * ============================================================================
 *
 */
global $cltd_example_db_version;
$cltd_example_db_version = '1.1'; // version changed from 1.0 to 1.1

/**
 * register_activation_hook implementation
 *
 * will be called when user activates plugin first time
 * must create needed database tables
 */
function cltd_example_install()
{
    global $wpdb;
    global $cltd_example_db_version;

    $table_name = $wpdb->prefix . 'banners_digi'; // do not forget about tables prefix

    // sql to create your table
    // NOTICE that:
    // 1. each field MUST be in separate line
    // 2. There must be two spaces between PRIMARY KEY and its name
    //    Like this: PRIMARY KEY[space][space](ID)
    // otherwise dbDelta will not work
    $sql = "CREATE TABLE " . $table_name . " (
      ID int(11) NOT NULL AUTO_INCREMENT,
      img_link varchar(68) NULL,
      banner_link varchar(68) NULL,
      PRIMARY KEY  (ID)
    );";

    // we do not execute sql directly
    // we are calling dbDelta which cant migrate database
    require_once(ABSPATH . 'wp-admin/includes/upgrade.php');
    dbDelta($sql);

    // save current database version for later use (on upgrade)
    add_option('cltd_example_db_version', $cltd_example_db_version);

    /**
     * [OPTIONAL] Example of updating to 1.1 version
     *
     * If you develop new version of plugin
     * just increment $cltd_example_db_version variable
     * and add following block of code
     *
     * must be repeated for each new version
     * to contain 200 chars rather 100 in version 1.0
     * and again we are not executing sql
     * we are using dbDelta to migrate table changes
     */
    $installed_ver = get_option('cltd_example_db_version');
    if ($installed_ver != $cltd_example_db_version) {
        $sql = "CREATE TABLE " . $table_name . " (
		  ID int(11) NOT NULL AUTO_INCREMENT,
		  img_link varchar(68) NULL,
		  banner_link varchar(68) NULL,
		  PRIMARY KEY  (ID)
        );";

        require_once(ABSPATH . 'wp-admin/includes/upgrade.php');
        dbDelta($sql);

        // notice that we are updating option, rather than adding it
        update_option('cltd_example_db_version', $cltd_example_db_version);
    }
}

register_activation_hook(__FILE__, 'cltd_example_install');

/**
 * register_activation_hook implementation
 *
 * [OPTIONAL]
 * additional implementation of register_activation_hook
 * to insert some dummy data
 */
function cltd_example_install_data()
{
    global $wpdb;

    $table_name = $wpdb->prefix . 'banners_digi'; // do not forget about tables prefix

    $wpdb->insert($table_name, array(
        'img_link' => 254,
        'banner_link' => 25
    ));
    $wpdb->insert($table_name, array(
        'img_link' => 458,
        'banner_link' => 22
    ));
}

register_activation_hook(__FILE__, 'cltd_example_install_data');

/**
 * Trick to update plugin database, see docs
 */
function cltd_example_update_db_check()
{
    global $cltd_example_db_version;
    if (get_site_option('cltd_example_db_version') != $cltd_example_db_version) {
        cltd_example_install();
    }
}

add_action('plugins_loaded', 'cltd_example_update_db_check');

/**
 * PART 2. Defining Custom Table List
 * ============================================================================
 *
 * In this part you are going to define custom table list class,
 * that will display your database records in nice looking table
 *
 * http://codex.wordpress.org/Class_Reference/WP_List_Table
 * http://wordpress.org/extend/plugins/custom-list-table-example/
 */

if (!class_exists('WP_List_Table')) {
    require_once(ABSPATH . 'wp-admin/includes/class-wp-list-table.php');
}

/**
 * Custom_Table_Example_List_Table class that will display our custom table
 * records in nice table
 */
class Custom_Table_Example_List_Table extends WP_List_Table
{
    /**
     * [REQUIRED] You must declare constructor and give some basic params
     */
    function __construct()
    {
        global $status, $page;

        parent::__construct(array(
            'singular' => 'person',
            'plural' => 'persons',
        ));
    }

    /**
     * [REQUIRED] this is a default column renderer
     *
     * @param $item - row (key, value array)
     * @param $column_name - string (key)
     * @return HTML
     */
    function column_default($item, $column_name)
    {
        return $item[$column_name];
    }

    /**
     * [OPTIONAL] this is example, how to render specific column
     *
     * method img_link must be like this: "column_[column_name]"
     *
     * @param $item - row (key, value array)
     * @return HTML
     */
    function column_age($item)
    {
        return '<em>' . $item['banner_link'] . '</em>';
    }

    /**
     * [OPTIONAL] this is example, how to render column with actions,
     * when you hover row "Edit | Delete" links showed
     *
     * @param $item - row (key, value array)
     * @return HTML
     */
    function column_name($item)
    {
        // links going to /admin.php?page=[your_plugin_page][&other_params]
        // notice how we used $_REQUEST['page'], so action will be done on curren page
        // also notice how we use $this->_args['singular'] so in this example it will
        // be something like &person=2
        $actions = array(
            'edit' => sprintf('<a href="?page=persons_form&ID=%s">%s</a>', $item['ID'], __('Edit', 'cltd_example')),
            'delete' => sprintf('<a href="?page=%s&action=delete&ID=%s">%s</a>', $_REQUEST['page'], $item['ID'], __('Delete', 'cltd_example')),
        );

        return sprintf('%s %s',
            $item['img_link'],
            $this->row_actions($actions)
        );
    }

    /**
     * [REQUIRED] this is how checkbox column renders
     *
     * @param $item - row (key, value array)
     * @return HTML
     */
    function column_cb($item)
    {
        return sprintf(
            '<input type="checkbox" name="ID[]" value="%s" />'.'</br><a href="?page=persons_form&ID='.$item['ID'].'">ویرایش</a>',
            $item['ID']
        );
    }

    /**
     * [REQUIRED] This method return columns to display in table
     * you can skip columns that you do not want to show
     * like content, or description
     *
     * @return array
     */
    function get_columns()
    {
        $columns = array(
            'cb' => '<input type="checkbox" />', //Render a checkbox instead of text
            'img_link' => __('لینک تصویر', 'cltd_example'),
            'banner_link' => __('لینک بنر', 'cltd_example'),
            'edit' => __('ویرایش', 'cltd_example'),
        );
        return $columns;
    }

    /**
     * [OPTIONAL] This method return columns that may be used to sort table
     * all strings in array - is column names
     * notice that true on img_link column means that its default sort
     *
     * @return array
     */
    function get_sortable_columns()
    {
        $sortable_columns = array(
            'img_link' => array('img_link', true),
            'banner_link' => array('banner_link', false),
            'edit' => array('banner_link', false),
        );
        return $sortable_columns;
    }

    /**
     * [OPTIONAL] Return array of bult actions if has any
     *
     * @return array
     */
    function get_bulk_actions()
    {
        $actions = array(
            'delete' => 'Delete',
            'edit' => 'Edit'
        );
        return $actions;
    }

    /**
     * [OPTIONAL] This method processes bulk actions
     * it can be outside of class
     * it can not use wp_redirect coz there is output already
     * in this example we are processing delete action
     * message about successful deletion will be shown on page in next part
     */
    function process_bulk_action()
    {
        global $wpdb;
        $table_name = $wpdb->prefix . 'banners_digi'; // do not forget about tables prefix

        if ('delete' === $this->current_action()) {
            $ids = isset($_REQUEST['ID']) ? $_REQUEST['ID'] : array();
            if (is_array($ids)) $ids = implode(',', $ids);

            if (!empty($ids)) {
                $wpdb->query("DELETE FROM $table_name WHERE ID IN($ids)");
            }
        }
    }

    /**
     * [REQUIRED] This is the most important method
     *
     * It will get rows from database and prepare them to be showed in table
     */
    function prepare_items()
    {
        global $wpdb;
        $table_name = $wpdb->prefix . 'banners_digi'; // do not forget about tables prefix

        $per_page = 5; // constant, how much records will be shown per page

        $columns = $this->get_columns();
        $hidden = array();
        $sortable = $this->get_sortable_columns();

        // here we configure table headers, defined in our methods
        $this->_column_headers = array($columns, $hidden, $sortable);

        // [OPTIONAL] process bulk action if any
        $this->process_bulk_action();

        // will be used in pagination settings
        $total_items = $wpdb->get_var("SELECT COUNT(ID) FROM $table_name");

        // prepare query params, as usual current page, order by and order direction
        $paged = isset($_REQUEST['paged']) ? max(0, intval($_REQUEST['paged'] - 1) * $per_page) : 0;
        $orderby = (isset($_REQUEST['orderby']) && in_array($_REQUEST['orderby'], array_keys($this->get_sortable_columns()))) ? $_REQUEST['orderby'] : 'img_link';
        $order = (isset($_REQUEST['order']) && in_array($_REQUEST['order'], array('asc', 'desc'))) ? $_REQUEST['order'] : 'asc';

        // [REQUIRED] define $items array
        // notice that last argument is ARRAY_A, so we will retrieve array
        $this->items = $wpdb->get_results($wpdb->prepare("SELECT * FROM $table_name ORDER BY $orderby $order LIMIT %d OFFSET %d", $per_page, $paged), ARRAY_A);

        // [REQUIRED] configure pagination
        $this->set_pagination_args(array(
            'total_items' => $total_items, // total items defined above
            'per_page' => $per_page, // per page constant defined at top of method
            'total_pages' => ceil($total_items / $per_page) // calculate pages count
        ));
    }
}

/**
 * PART 3. Admin page
 * ============================================================================
 *
 * In this part you are going to add admin page for custom table
 *
 * http://codex.wordpress.org/Administration_Menus
 */

/**
 * admin_menu hook implementation, will add pages to list persons and to add new one
 */
function cltd_example_admin_menu()
{
    add_menu_page(__('لیست بنرها', 'cltd_example'), __('لیست بنرها', 'cltd_example'), 'activate_plugins', 'persons', 'cltd_example_persons_page_handler');
    add_submenu_page('persons', __('لیست بنرها', 'cltd_example'), __('لیست بنرها', 'cltd_example'), 'activate_plugins', 'persons', 'cltd_example_persons_page_handler');
    // add new will be described in next part
    add_submenu_page('persons', __('افزودن بنر', 'cltd_example'), __('افزودن بنر', 'cltd_example'), 'activate_plugins', 'persons_form', 'cltd_example_persons_form_page_handler');
}

add_action('admin_menu', 'cltd_example_admin_menu');

/**
 * List page handler
 *
 * This function renders our custom table
 * Notice how we display message about successfull deletion
 * Actualy this is very easy, and you can add as many features
 * as you want.
 *
 * Look into /wp-admin/includes/class-wp-*-list-table.php for examples
 */
function cltd_example_persons_page_handler()
{
    global $wpdb;

    $table = new Custom_Table_Example_List_Table();
    $table->prepare_items();

    $message = '';
    if ('delete' === $table->current_action()) {
        $message = '<div class="updated below-h2" id="message"><p>' . sprintf(__('موارد حذف شده %d', 'cltd_example'), count($_REQUEST['ID'])) . '</p></div>';
    }
    ?>
<div class="wrap">

    <div class="icon32 icon32-posts-post" id="icon-edit"><br></div>
    <h2><?php _e('لیست بنرها', 'cltd_example')?> <a class="add-new-h2"
                                 href="<?php echo get_admin_url(get_current_blog_id(), 'admin.php?page=persons_form');?>"><?php _e('افزودن بنر', 'cltd_example')?></a>
    </h2>
    <?php echo $message; ?>

    <form id="persons-table" method="GET">
        <input type="hidden" name="page" value="<?php echo $_REQUEST['page'] ?>"/>
        <?php $table->display() ?>
    </form>

</div>
<?php
}

/**
 * PART 4. Form for adding andor editing row
 * ============================================================================
 *
 * In this part you are going to add admin page for adding andor editing items
 * You cant put all form into this function, but in this example form will
 * be placed into meta box, and if you want you can split your form into
 * as many meta boxes as you want
 *
 * http://codex.wordpress.org/Data_Validation
 */

/**
 * Form page handler checks is there some data posted and tries to save it
 * Also it renders basic wrapper in which we are callin meta box render
 */
function cltd_example_persons_form_page_handler()
{
    global $wpdb;
    $table_name = $wpdb->prefix . 'banners_digi'; // do not forget about tables prefix

    $message = '';
    $notice = '';

    // this is default $item which will be used for new records
    $default = array(
        'ID' => 0,
        'img_link' => '',
        'banner_link' => null,
    );

    // here we are verifying does this request is post back and have correct nonce
    if ( isset($_REQUEST['nonce']) && wp_verify_nonce($_REQUEST['nonce'], basename(__FILE__))) {
        // combine our default item with request params
        $item = shortcode_atts($default, $_REQUEST);
        // validate data, and if all ok save item to database
        // if ID is zero insert otherwise update
        $item_valid = cltd_example_validate_person($item);
        if ($item_valid === true) {
            if ($item['ID'] == 0) {
                $result = $wpdb->insert($table_name, $item);
                $item['ID'] = $wpdb->insert_id;
                if ($result) {
                    $message = __('بنر با موفقیت ذخیره شد', 'cltd_example');
                } else {
                    $notice = __('مشکلی در ذخیره به وجود آمد', 'cltd_example');
                }
            } else {
                $result = $wpdb->update($table_name, $item, array('ID' => $item['ID']));
                if ($result) {
                    $message = __('آیتم با موفقیت بروز شد.', 'cltd_example');
                } else {
                    $notice = __('مشکلی حین بروزرسانی اتفاق افتاد', 'cltd_example');
                }
            }
        } else {
            // if $item_valid not true it contains error message(s)
            $notice = $item_valid;
        }
    }
    else {
        // if this is not post back we load item to edit or give new one to create
        $item = $default;
        if (isset($_REQUEST['ID'])) {
            $item = $wpdb->get_row($wpdb->prepare("SELECT * FROM $table_name WHERE ID = %d", $_REQUEST['ID']), ARRAY_A);
            if (!$item) {
                $item = $default;
                $notice = __('Item not found', 'cltd_example');
            }
        }
    }

    // here we adding our custom meta box
    add_meta_box('persons_form_meta_box', 'اطلاعات بنر', 'cltd_example_persons_form_meta_box_handler', 'person', 'normal', 'default');

    ?>
<div class="wrap">
    <div class="icon32 icon32-posts-post" id="icon-edit"><br></div>
    <h2><?php _e('بنر', 'cltd_example')?> <a class="add-new-h2"
                                href="<?php echo get_admin_url(get_current_blog_id(), 'admin.php?page=persons');?>"><?php _e('بازگشت به لیست', 'cltd_example')?></a>
    </h2>

    <?php if (!empty($notice)): ?>
    <div id="notice" class="error"><p><?php echo $notice ?></p></div>
    <?php endif;?>
    <?php if (!empty($message)): ?>
    <div id="message" class="updated"><p><?php echo $message ?></p></div>
    <?php endif;?>

    <form id="form" method="POST">
        <input type="hidden" name="nonce" value="<?php echo wp_create_nonce(basename(__FILE__))?>"/>
        <?php /* NOTICE: here we storing ID to determine will be item added or updated */ ?>
        <input type="hidden" name="ID" value="<?php echo $item['ID'] ?>"/>

        <div class="metabox-holder" id="poststuff">
            <div id="post-body">
                <div id="post-body-content">
                    <?php /* And here we call our custom meta box */ ?>
                    <?php do_meta_boxes('person', 'normal', $item); ?>
                    <input type="submit" value="<?php _e('Save', 'cltd_example')?>" id="submit" class="button-primary" name="submit">
                </div>
            </div>
        </div>
    </form>
</div>
<?php
}

/**
 * This function renders our custom meta box
 * $item is row
 *
 * @param $item
 */
function cltd_example_persons_form_meta_box_handler($item)
{
	wp_enqueue_script('jquery');
// This will enqueue the Media Uploader script
wp_enqueue_media();

    ?>

<table cellspacing="2" cellpadding="5" style="width: 100%;" class="form-table">
    <tbody>
    <tr class="form-field">
        <th valign="top" scope="row">
            <label for="img_link"><?php _e('لینک تصویر', 'cltd_example')?></label>
        </th>
        <td>
            <input id="img_link" name="img_link" type="text" style="width: 95%"  value="<?php echo esc_attr($item['img_link'])?>"
                   class="code" placeholder="<?php _e('لینک تصویر را وارد کنید', 'cltd_example')?>" >
				   
				       <input type="button" name="upload-btn" id="upload-btn" class="button-secondary" value="بارگذاری تصویر">
<script type="text/javascript">
jQuery(document).ready(function($){
    $('#upload-btn').click(function(e) {
        e.preventDefault();
        var image = wp.media({ 
            title: 'Upload Image',
            // mutiple: true if you want to upload multiple files at once
            multiple: false
        }).open()
        .on('select', function(e){
            // This will return the selected image from the Media Uploader, the result is an object
            var uploaded_image = image.state().get('selection').first();
            // We convert uploaded_image to a JSON object to make accessing it easier
            // Output to the console uploaded_image
            console.log(uploaded_image);
            var img_link = uploaded_image.toJSON().url;
            // Let's assign the url value to the input field
            $('#img_link').val(img_link);
        });
    });
});
</script>
				   
				   
        </td>
    </tr>
    <tr class="form-field">
        <th valign="top" scope="row">
            <label for="banner_link"><?php _e('لینک بنر', 'cltd_example')?></label>
        </th>
        <td>
            <input id="banner_link" name="banner_link" type="text" style="width: 95%" value="<?php echo esc_attr($item['banner_link'])?>"
                   class="code" placeholder="<?php _e('لینک بنر را وارد کنید', 'cltd_example')?>" >
        </td>
    </tr>
    </tbody>
</table>
<?php
}

/**
 * Simple function that validates data and retrieve bool on success
 * and error message(s) on error
 *
 * @param $item
 * @return bool|string
 */
function cltd_example_validate_person($item)
{
    $messages = array();

    if (empty($item['img_link'])) $messages[] = __('Image is required', 'cltd_example');
    if (empty($item['banner_link'])) $messages[] = __('Target link is required', 'cltd_example');

    if (empty($messages)) return true;
    return implode('<br />', $messages);
}

/**
 * Do not forget about translating your plugin, use __('english string', 'your_uniq_plugin_name') to retrieve translated string
 * and _e('english string', 'your_uniq_plugin_name') to echo it
 * in this example plugin your_uniq_plugin_name == cltd_example
 *
 * to create translation file, use poedit FileNew catalog...
 * Fill img_link of project, add "." to path (ENSURE that it was added - must be in list)
 * and on last tab add "__" and "_e"
 *
 * Name your file like this: [my_plugin]-[ru_RU].po
 *
 * http://codex.wordpress.org/Writing_a_Plugin#Internationalizing_Your_Plugin
 * http://codex.wordpress.org/I18n_for_WordPress_Developers
 */
function cltd_example_languages()
{
    load_plugin_textdomain('cltd_example', false, dirname(plugin_basename(__FILE__)));
}

add_action('init', 'cltd_example_languages');

///////////////////////
 function show_banners_odd($atts) {
		global $wpdb;
		        $table_name = $wpdb->prefix . 'banners_digi'; // do not forget about tables prefix
					 $sql = $wpdb->get_results( 
						"
						SELECT * 
						FROM $table_name
						"
						);
		
		foreach($sql as $res)
			echo '<img  width="100%"  src="'.$res->img_link.'"  href="'.$res->banner_link.'"    target="_blank" />';

 }
add_shortcode('show-banners-odd', 'show_banners_odd');
