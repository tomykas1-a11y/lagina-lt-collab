# Gemini theme-mod migration — REVIEW MATERIAL ONLY

> **DANGER — Do not execute, deploy, merge, or activate.** This is an untested draft supplied in the ChatGPT/Gemini collaboration chat for Copilot's independent analysis. It is deliberately stored as **Markdown, not a runnable PHP file**. It is **not approved** for staging or production.

## Target and review context

- WordPress / ListingHive marketplace at Lagina.lt.
- Parent theme: `listinghive`.
- Proposed child theme: `listinghive-child`.
- Theme scaffold: [PR #1](https://github.com/tomykas1-a11y/lagina-lt-collab/pull/1).
- This migration was **not included in PR #1**, so Copilot could not review it in GitHub Task `8504bb65-78c4-4749-9ff9-90594272ad63`.
- The code below is the latest **Gemini-authored draft** submitted for independent review, not a tested implementation.

## Gemini-proposed `migrate-theme-mods.php`

```php
<?php
/**
 * Safe Theme Mod and Custom CSS Migration Script for Lagina.lt
 * Execution: wp eval-file migrate-theme-mods.php
 */

// CONFIGURATION & SAFETY LOCKS
$is_preflight     = true;  // Set to false ONLY after Copilot review & approval.
$backup_confirmed = false; // Set to true ONLY after verifying a staging backup.

$parent_theme_slug = 'listinghive';
$child_theme_slug  = 'listinghive-child';

echo "=================================================\n";
echo "LAGINA.LT - THEME MIGRATION SCRIPT\n";
echo "=================================================\n";

if ( $is_preflight ) {
    echo "[PREFLIGHT MODE] - No database changes will be made.\n\n";
}

// 1. ENVIRONMENT & STATE VALIDATION
$env_type = wp_get_environment_type();
if ( 'production' === $env_type ) {
    die( "FATAL: This script is restricted from running on the production environment.\n" );
}

if ( ! $is_preflight && ! $backup_confirmed ) {
    die( "FATAL: Execution locked. A restorable backup must be confirmed (\$backup_confirmed = true).\n" );
}

$current_theme = wp_get_theme();
if ( $current_theme->get_stylesheet() !== $parent_theme_slug ) {
    die( "FATAL: The active theme must be '{$parent_theme_slug}' before running this script.\n" );
}

$child_theme_obj = wp_get_theme( $child_theme_slug );
if ( ! $child_theme_obj->exists() ) {
    die( "FATAL: The child theme '{$child_theme_slug}' is not installed in wp-content/themes/.\n" );
}

// 2. IDEMPOTENCY & EXISTENCE CHECKS
$parent_mods = get_option( "theme_mods_{$parent_theme_slug}" );
if ( ! is_array( $parent_mods ) ) {
    die( "FATAL: No parent theme mods found. Migration aborted.\n" );
}

$child_mods = get_option( "theme_mods_{$child_theme_slug}" );
$skip_mods  = false;
if ( is_array( $child_mods ) && ! empty( $child_mods ) ) {
    echo "NOTICE: Child theme mods already exist. Skipping mods migration to prevent overwrite.\n";
    $skip_mods = true;
}

$child_css_post = wp_get_custom_css_post( $child_theme_slug );
$skip_css       = false;
if ( $child_css_post instanceof WP_Post ) {
    echo "NOTICE: Child custom CSS post already exists. Skipping CSS migration to prevent overwrite.\n";
    $skip_css = true;
}

if ( $skip_mods && $skip_css ) {
    echo "\nSUCCESS: All data previously migrated. Safe to rerun, but nothing to do.\n";
    exit;
}

// 3. EXECUTION LOGIC
// A. Migrate Theme Mods
if ( ! $skip_mods ) {
    echo "[PREPARING] Theme Mods Migration...\n";

    // Deep copy to prevent any PHP reference artifacts.
    $new_child_mods = unserialize( serialize( $parent_mods ) );

    if ( isset( $new_child_mods['custom_css_post_id'] ) ) {
        unset( $new_child_mods['custom_css_post_id'] );
        echo " - Parent custom_css_post_id safely stripped from array.\n";
    }

    if ( ! $is_preflight ) {
        $result = update_option( "theme_mods_{$child_theme_slug}", $new_child_mods );
        echo $result ? " -> SUCCESS: Theme mods migrated to child namespace.\n" : " -> ERROR: Failed to write theme mods.\n";
    } else {
        echo " -> (Preflight) Would create theme_mods_{$child_theme_slug} option.\n";
    }
}

// B. Migrate Additional CSS
if ( ! $skip_css ) {
    echo "\n[PREPARING] Additional CSS Migration...\n";
    $parent_css_post = wp_get_custom_css_post( $parent_theme_slug );

    if ( $parent_css_post instanceof WP_Post && ! empty( $parent_css_post->post_content ) ) {
        $raw_css = $parent_css_post->post_content;

        if ( ! $is_preflight ) {
            $css_update = wp_update_custom_css_post(
                $raw_css,
                array( 'stylesheet' => $child_theme_slug )
            );

            if ( ! is_wp_error( $css_update ) ) {
                echo " -> SUCCESS: CSS cloned safely. New post ID automatically linked to child theme.\n";
            } else {
                echo " -> ERROR: CSS Migration Failed: " . $css_update->get_error_message() . "\n";
            }
        } else {
            echo " -> (Preflight) Would create unique CSS post (" . strlen( $raw_css ) . " bytes) assigned to {$child_theme_slug}.\n";
        }
    } else {
        echo " -> NOTICE: No Additional CSS found in parent theme.\n";
    }
}

echo "\n=================================================\n";
if ( $is_preflight ) {
    echo "PREFLIGHT COMPLETE. No changes made.\n";
} else {
    echo "MIGRATION COMPLETE. You may now activate the child theme.\n";
}
echo "=================================================\n";
```

## Gemini's proposed test plan (review, not authorization)

1. **Guards:** Simulate production environment, missing backup acknowledgment, and wrong active theme in an **isolated test harness**. Do **not** change the live staging theme just to test a guard.
2. **Preflight:** Check that dry run performs no writes to options or `custom_css` posts.
3. **Execution/idempotency:** With synthetic data in a disposable WordPress instance, inspect parent and child options/CSS posts and run twice without data loss or duplicate posts.
4. **Rollback:** In disposable environment verify parent mods and parent CSS remain intact through switch and rollback.

## Mandatory independent-review questions

- Does `wp_get_environment_type()` establish staging identity? What if `WP_ENVIRONMENT_TYPE` is misconfigured? Require an exact expected `home`/`siteurl` staging host and WP-CLI guard.
- Does `! empty($child_mods)` protect an existing empty array or non-array option? Distinguish absent option from any existing option.
- Does `wp_update_custom_css_post()` set `custom_css_post_id` when the target child theme is **inactive**? Inspect WordPress source and test.
- Are all failures nonzero WP-CLI exit statuses? Does writing mods before failed CSS creation cause a partial migration while still printing "MIGRATION COMPLETE"?
- Is `unserialize( serialize(...) )` necessary or safe for values that may contain PHP objects? Prefer preserving original arrays with copy-on-write.
- Is the backup merely asserted by an editable Boolean, and how is a real restorable backup verified?
- Is preflight's output unambiguous when child mods or CSS already exist, when parent CSS is empty, or when parent option is missing?
- Can settings or Additional CSS have theme-specific assumptions that require staging visual checks?
- Is the rollback truly non-destructive and free from raw-SQL mutation?
- Does the proposed plan avoid changing staging themes to test guards?
- Report actual PHP version, static/dynamic test evidence, and any remaining unverified behavior.

**Reviewer deliverable:** PASS/FAIL by issue; corrected implementation only as a proposed diff or separate branch; DO NOT deploy, activate, or merge any theme or migration. Parent `lagina-lt` repo remains private. Payments disabled; Montonio inactive.
