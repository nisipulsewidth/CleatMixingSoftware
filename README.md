# CleatMixingSoftware
This is a max patch (and a sub-patch) I developed as mixing software for Dani Jean-Baptiste's CLEAT Series performance on June 14th, 2024. The CLEAT System is a 16-channel audio system at Elastic Arts created for the composition and performance of spatial music, with each of the 16 speakers arranged in a 4x4 matrix. The max patches are an adaptation and expansion of the sample max patches created for the CLEAT system by Stephan Moore, available at "https://cleat.info/resources/". I'm mostly uploading this patch to show Stephan and Anais, the latter of whom also developed some max patches for a CLEAT performance.

A little bit of background: over the past three years or so, Dani Jean-Baptiste has been composing, recording, and mixing a 90-minute sound collage. It'll probably get released some time soon; I'll update this GitHub file whenever that happens. Dani approached me about helping them do a performance of said sound collage as part of Elastic's CLEAT series in November of 2023, which I happily agreed to help them with. In that time since, we have composed and choreographed dance/performance art for a ~45 minute condensed version of the sound collage. This max patch was created specifically for the mixing and playing of that sound collage.

cleatMixingSoftware.maxpat is the software/interface used for playing 37 wav files during the performance. These files are:

	> 7 standard audio tracks (channel[1,...,7].wav)
	> 6 audio tracks that rotate counter-clockwise around the CLEAT speaker matrix (rotator1Channel[1,...,6].wav)
	> 7 audio tracks that rotate clockwise around the CLEAT speaker matrix (rotator2Channel[1,...,7].wav)
	> 1 audio track that plays uniformly across all 16 speakers (specialCase.wav)
	> 16 audio tracks, each of which plays on a dedicated CLEAT speaker (voice[1,...,16].wav)

It should be noted that each of the audio tracks are consolidated mixes from the 85 audio tracks from the larger sound collage logic project. As such, no individual audio file plays continuously through the piece, and there is no point where all 37 tracks are playing at the same time. All files are in stereo, but are converted to mono by summing L+R and halving the amplitude of the sum.

The voice & specialCase files each have their own gain bar towards the bottom of the presentation mode. Each of the voice[1,...,16] files only play during a 2-minute segment where cacophony is emphasized in the composition. specialCase.wav only plays at two points: for a 3-minute monologue ~10 minutes into the piece, and once for a short burst to indicate the end of the aforementioned cacophonic section.

The remainder of the audio files are each controlled by a 192x192 spatialization matrix using the pictslider~ object in max. The point on the matrix represents the "physical location" of each track. Each matrix directly maps to the CLEAT system facing the projection wall. Speaker #n (n = [1,...,16]) are each represented in the matrix at points (64*((n-1)%4), 192-64*⌊(n-1)/4⌋). Each matrix also has a slope element, which I'll get into more in the following paragraph, but just know that the higher the slope value, the more speakers the audio track plays out of, expanded radially outward from the point. The rotator tracks also have two other variables: the rotational frequency of the rotating audio tracks, which ranges from 0-0.5hZ, and the distance from the center of the matrix.

yAltogether~.maxpat is a sub-patch invoked in cleatMixingSoftware that takes each spatialized audio track and converts it to appropriately scaled signals across all 16 speakers. yAltogether~ takes the point coordinates at its respective spatialization matrix and generates 16 vector magnitudes, those magnitudes being the distance from the point to each of the 16 speakers. These magnitudes, and the slope input, are then used to generate an amplitude modifier of the input audio track to be output at each speaker, with the amplitude modifier being inversely correlated with the vector magnitude. The amplitude modifier maxes out at 1 and mins out at 0. Think of it as a cone moving across the space, with the vertex of the cone located at the input coordinates and slope controlling the size of the cone's round base. The amplitude at each speaker is equivalent to the height of the cone at the speaker-point. In future implementations of this, I would like to have the modifier be based on decibels, not signal amplitude, but I didn't have the time to bug/field test that implementation.

As it stands right now, the software contains two glitches/poorly-implemented functions. The first is that in yAltogether~, I utilize the "scale" max object to calculate the amplitude modifier, with slope controlling the output min/max. The problem with this is that "scale" does not refresh its output when the scaling parameters are altered, but when the number to be scaled, in this case the vector magnitude, is altered. This means that only moving the slope slider in cleatMixingSoftware does not change how many speakers the audio file plays through; you have to first move the slope slider THEN jiggle the point in the matrix. The second poorly implemented function has to do with the slider at the bottom of the presentation mode. Max/MSP doesn't really have a good way to scrub across multiple audio files, so what I sufficed with was a slider that would send a seek message to each sfplay~ object that the audio files were loaded into, as well as a pause and resume button attached to each sfplay~ object. This proved reliable enough, but inconsistent and buggy. Like sometimes after pressing resume after pressing pause, it just wouldn't play, but I figured this was fine, as I'm only using those 3 objects for rehearsing with Dani.
