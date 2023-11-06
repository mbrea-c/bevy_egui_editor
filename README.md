### bevy_egui_inspector

This repository is simply a neat repackaging of the [dock example from
bevy-inspector-egui](https://github.com/jakobhellermann/bevy-inspector-egui/blob/95bb3940529c84179a60a5b7c49a2587e85f1c22/crates/bevy-inspector-egui/examples/integrations/egui_dock.rs)
as a library crate for ease of use.

#### Usage

Add the dependency to your project:

```bash
cargo add bevy_egui_inspector
```

Add the `bevy_egui_inspector::EguiEditorPlugin` to your `App` and the
`EditorCamera` marker struct to your `Camera`:

```rust
// Example adapted from
// https://github.com/jakobhellermann/bevy-inspector-egui/blob/95bb3940529c84179a60a5b7c49a2587e85f1c22/crates/bevy-inspector-egui/examples/integrations/egui_dock.rs
extern crate bevy_egui_editor;

use bevy::prelude::*;
use bevy_egui_editor::EditorCamera;
use bevy_egui_editor::EguiEditorPlugin;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(EguiEditorPlugin)
        .add_systems(Startup, setup)
        .run();
}

fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    let box_size = 2.0;
    let box_thickness = 0.15;
    let box_offset = (box_size + box_thickness) / 2.0;

    // left - red
    let mut transform = Transform::from_xyz(-box_offset, box_offset, 0.0);
    transform.rotate(Quat::from_rotation_z(std::f32::consts::FRAC_PI_2));
    commands.spawn(PbrBundle {
        mesh: meshes.add(Mesh::from(shape::Box::new(
            box_size,
            box_thickness,
            box_size,
        ))),
        transform,
        material: materials.add(StandardMaterial {
            base_color: Color::rgb(0.63, 0.065, 0.05),
            ..Default::default()
        }),
        ..Default::default()
    });
    // right - green
    let mut transform = Transform::from_xyz(box_offset, box_offset, 0.0);
    transform.rotate(Quat::from_rotation_z(std::f32::consts::FRAC_PI_2));
    commands.spawn(PbrBundle {
        mesh: meshes.add(Mesh::from(shape::Box::new(
            box_size,
            box_thickness,
            box_size,
        ))),
        transform,
        material: materials.add(StandardMaterial {
            base_color: Color::rgb(0.14, 0.45, 0.091),
            ..Default::default()
        }),
        ..Default::default()
    });
    // bottom - white
    commands.spawn(PbrBundle {
        mesh: meshes.add(Mesh::from(shape::Box::new(
            box_size + 2.0 * box_thickness,
            box_thickness,
            box_size,
        ))),
        material: materials.add(StandardMaterial {
            base_color: Color::rgb(0.725, 0.71, 0.68),
            ..Default::default()
        }),
        ..Default::default()
    });
    // top - white
    let transform = Transform::from_xyz(0.0, 2.0 * box_offset, 0.0);
    commands.spawn(PbrBundle {
        mesh: meshes.add(Mesh::from(shape::Box::new(
            box_size + 2.0 * box_thickness,
            box_thickness,
            box_size,
        ))),
        transform,
        material: materials.add(StandardMaterial {
            base_color: Color::rgb(0.725, 0.71, 0.68),
            ..Default::default()
        }),
        ..Default::default()
    });
    // back - white
    let mut transform = Transform::from_xyz(0.0, box_offset, -box_offset);
    transform.rotate(Quat::from_rotation_x(std::f32::consts::FRAC_PI_2));
    commands.spawn(PbrBundle {
        mesh: meshes.add(Mesh::from(shape::Box::new(
            box_size + 2.0 * box_thickness,
            box_thickness,
            box_size + 2.0 * box_thickness,
        ))),
        transform,
        material: materials.add(StandardMaterial {
            base_color: Color::rgb(0.725, 0.71, 0.68),
            ..Default::default()
        }),
        ..Default::default()
    });

    // ambient light
    commands.insert_resource(AmbientLight {
        color: Color::WHITE,
        brightness: 0.02,
    });
    // top light
    commands
        .spawn(PbrBundle {
            mesh: meshes.add(Mesh::from(shape::Plane::from_size(0.4))),
            transform: Transform::from_matrix(Mat4::from_scale_rotation_translation(
                Vec3::ONE,
                Quat::from_rotation_x(std::f32::consts::PI),
                Vec3::new(0.0, box_size + 0.5 * box_thickness, 0.0),
            )),
            material: materials.add(StandardMaterial {
                base_color: Color::WHITE,
                emissive: Color::WHITE * 100.0,
                ..Default::default()
            }),
            ..Default::default()
        })
        .with_children(|builder| {
            builder.spawn(PointLightBundle {
                point_light: PointLight {
                    color: Color::WHITE,
                    shadows_enabled: true,
                    intensity: 25.0,
                    ..Default::default()
                },
                transform: Transform::from_translation((box_thickness + 0.05) * Vec3::Y),
                ..Default::default()
            });
        });
    // directional light
    commands.spawn(DirectionalLightBundle {
        directional_light: DirectionalLight {
            illuminance: 10000.0,
            ..default()
        },
        transform: Transform::from_rotation(Quat::from_rotation_x(-std::f32::consts::PI / 2.0)),
        ..Default::default()
    });

    // camera
    commands.spawn((
        Camera3dBundle {
            transform: Transform::from_xyz(0.0, box_offset, 4.0)
                .looking_at(Vec3::new(0.0, box_offset, 0.0), Vec3::Y),
            ..Default::default()
        },
        EditorCamera,
    ));
}
```
