# Canonical Data Model Proposal (Prototype)

_Last updated: 2026-03-01_

## Design Principles

- Keep **breed-level** and **individual-dog-level** data separate.
- Preserve **source provenance** for every field.
- Support uncertain/incomplete shelter data.
- Allow iterative enrichment as new APIs/sources are added.

## Entity Overview

## 1) `Breed`
Core breed profile.

Suggested fields:
- `breed_id` (uuid)
- `canonical_name`
- `aliases` (json array)
- `group_registry` (AKC/CKC/FCI mappings)
- `size_min_kg`, `size_max_kg`
- `height_min_cm`, `height_max_cm`
- `life_expectancy_min_years`, `life_expectancy_max_years`
- `coat_type`, `coat_length_score`, `shedding_score`, `grooming_score`
- `energy_score`, `trainability_score`, `barking_score`, `protectiveness_score`
- `good_with_children_score`, `good_with_dogs_score`, `good_with_strangers_score`
- `temperament_text` (curated summary)
- `source_confidence`
- `updated_at`

## 2) `BreedHealthProfile`
Breed-specific health and screening metadata.

Suggested fields:
- `breed_health_id` (uuid)
- `breed_id` (fk)
- `condition_name`
- `screening_type` (phenotypic / DNA / specialist exam)
- `recommended_screening_age`
- `program_reference` (e.g., CHIC requirement ref)
- `severity_weight`
- `source_name`, `source_url`
- `updated_at`

## 3) `Organization`
Shelter, rescue, breeder, registry listing organization.

Suggested fields:
- `organization_id` (uuid)
- `org_type` (`shelter`, `rescue`, `breeder`, `registry`, `aggregator`)
- `display_name`
- `country`, `province_state`, `city`, `postal_code`
- `website_url`, `contact_email`, `contact_phone`
- `accreditations` (json)
- `source_platform`

## 4) `DogListing`
Individual adoptable/purchase listing.

Suggested fields:
- `listing_id` (uuid)
- `source_platform` (Petfinder/RescueGroups/CKC Puppy List/AKC Marketplace/etc.)
- `source_listing_id`
- `organization_id` (fk)
- `acquisition_channel` (`adoption`, `purchase`)
- `status` (`available`, `pending`, `adopted`, `unknown`)
- `name`
- `sex`
- `age_category`
- `estimated_birth_date` (nullable)
- `primary_breed_id` (nullable fk)
- `secondary_breed_id` (nullable fk)
- `breed_text_raw`
- `description_raw`
- `size_class`
- `weight_kg` (nullable)
- `spayed_neutered` (nullable bool)
- `house_trained` (nullable bool)
- `good_with_children` (nullable enum/score)
- `good_with_dogs` (nullable enum/score)
- `good_with_cats` (nullable enum/score)
- `special_needs_flags` (json)
- `adoption_fee_amount` (nullable)
- `currency`
- `listing_url`
- `first_seen_at`, `last_seen_at`

## 5) `ListingMedia`
Images/videos and optional hashes for dedupe.

Suggested fields:
- `media_id` (uuid)
- `listing_id` (fk)
- `media_type`
- `media_url`
- `image_hash` (nullable)
- `is_primary`

## 6) `SourceAttribution`
Field-level provenance.

Suggested fields:
- `attribution_id` (uuid)
- `entity_type`, `entity_id`
- `field_name`
- `source_name`
- `source_url`
- `ingested_at`
- `license_status` (`allowed`, `restricted`, `unknown`)
- `confidence`

## 7) `OwnerPreferenceProfile`
User-side matching inputs.

Suggested fields:
- `profile_id` (uuid)
- `household_children_age_bands` (json)
- `other_pets` (json)
- `home_type` (apartment/house/rural)
- `activity_level`
- `allergy_sensitivity`
- `grooming_tolerance`
- `noise_tolerance`
- `experience_level`
- `budget_range`
- `location`, `max_travel_distance_km`
- `acquisition_preference` (`adoption`, `purchase`, `either`)

## 8) `MatchScore`
Transparent score components.

Suggested fields:
- `match_id` (uuid)
- `profile_id` (fk)
- `target_type` (`breed`, `dog_listing`)
- `target_id`
- `overall_score` (0-100)
- `score_breakdown` (json: temperament fit, exercise fit, household fit, health-risk fit, logistics fit)
- `explanation_text`
- `computed_at`

## Minimal v1 Table Set

If you want to move very fast:

- `Breed`
- `DogListing`
- `Organization`
- `SourceAttribution`
- `OwnerPreferenceProfile`
- `MatchScore`

## Normalization Notes

- Store raw categorical text and mapped enums together.
- Never overwrite raw source values.
- Add a source reliability weight per platform.
- Track data staleness for listings (e.g., if not refreshed in X days).

## Matching Strategy (v1)

1. Score breed-level suitability first.
2. Re-rank by live listing availability near the user.
3. Apply compatibility hard filters:
   - mandatory children compatibility
   - mandatory cat/dog compatibility
   - acquisition channel preference
4. Present explanation-first output (why this match, why not others).
