You're starting to write similar code project after project and think it would be useful to establish some base components that can be reused across multiple projects.  First you create a library module to isolate your code.  Then you need to figure out how you're going to include it in each project.  

If it's open source, hosting it in a public maven repository like JCenter/Maven Central is a good solution.  If the code can't be shared publicly, the simplest way I've found is this:


[[MORE]]


## Step 1
Copy the packaged library file:

	MyReusableLibrary/build/outputs/aar/MyReusableLibrary-release.aar

and paste it into the `app/libs` directory of each project you want to use it in.


## Step 2
Add the following to the `app/build.gradle` of each project

	repositories {
	    ...
	    flatDir {
	        dirs 'libs'
	    }
	}
	
	dependencies {
		...
		compile ':MyReusableLibrary-release.aar
	}


## What I like about it
It's self-contained for library consumers.  If a developer tries to download and build a project that uses this library, there are no additional commands to run and no extra credentials to track down.

It's low maintenance for library providers.  You can host the aar files directly in your source repository or wherever else makes sense.  You don't need to worry about maintaining a private maven server.  


## What I don't like about it
It doesn't include transitive dependencies automatically.  Since there is no associated pom file, you need to describe them along with your library.  If the example library above depended on the Android Support Library, the project dependencies would need to include both:

	
	dependencies {
		...
		compile ':MyReusableLibrary-release.aar
		compile 'com.android.support:appcompat-v7:24.2.0'
	}



## Additional tips
Add the following to the `build.gradle` of your library module to store versioned aar files:

	android {
		defaultConfig {
			versionName "1.0"
		}
	    buildTypes {
	        release {
	            archivesBaseName = "${project.name}-${android.defaultConfig.versionName}"
	        }
	    }
	}
	
	task copyAar(type: Copy) {
	    from('build/outputs/aar')
	    into('../app/libs')
	    include(archivesBaseName + '-release.aar')
	}
	copyAar.dependsOn assemble


Running `./gradlew copyAar` assembles the file `MyResuableLibrary-1.0-release.aar` and copies it to the libs folder.

Have a sample app module in your library project so you can test your library before using it in other projects.  See the [Picasso library](https://github.com/square/picasso) from Square (or many of the other Android library projecst on Github) as an example.  The /picasso directory is the library and /picasso-sample directory is the sample app.  You can also use this sample app module to test the flatDir dependency approach described above.
