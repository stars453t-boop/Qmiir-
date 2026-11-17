using UnityEngine;

public class DynamicActionCamera : MonoBehaviour
{
    public Transform player;
    public Rigidbody playerRb;
    public Transform aimTarget; // Put an empty object on chest for ADS

    [Header("Camera Modes")]
    public Vector3 groundOffset = new Vector3(0.6f, 1.7f, -2.5f);
    public Vector3 adsOffset = new Vector3(0.45f, 1.6f, -0.8f); // COD ADS
    public Vector3 skydiveOffset = new Vector3(0f, 2.0f, -4.5f);
    public Vector3 vehicleOffset = new Vector3(0f, 3.5f, -7f);
    
    public bool isSkydiving = false;
    public bool isAiming = false;
    public bool isInVehicle = false;
    public bool isBossNearby = false; // Turn on in final level

    [Header("Speed & FOV Effects")]
    public float baseFOV = 60f;
    public float adsFOV = 45f;
    public float maxSkydiveFOV = 95f;
    public float shakeIntensity = 0.15f;

    private Camera cam;
    private Vector3 currentOffset;

    void Start()
    {
        cam = GetComponent<Camera>();
        currentOffset = groundOffset;
    }

    void LateUpdate()
    {
        if (!player) return;

        // 1. Pick mode - Priority: Skydive > Vehicle > ADS > Ground
        Vector3 targetOffset = groundOffset;
        if (isSkydiving) targetOffset = skydiveOffset;
        else if (isInVehicle) targetOffset = vehicleOffset;
        else if (isAiming) targetOffset = adsOffset;

        // Smooth offset change - no snapping
        currentOffset = Vector3.Lerp(currentOffset, targetOffset, Time.deltaTime * 8f);
        Vector3 desiredPosition = player.position + player.TransformDirection(currentOffset);

        // 2. Shake - Wind in skydive + Boss fear shake
        float shake = 0f;
        if (isSkydiving) shake = shakeIntensity;
        if (isBossNearby) shake += 0.08f + Mathf.Sin(Time.time * 20f) * 0.05f; // Kraken / NYX shake

        desiredPosition += Random.insideUnitSphere * shake;

        transform.position = Vector3.Lerp(transform.position, desiredPosition, Time.deltaTime * 12f);
        
        // Look at player chest, or aim target when ADS
        Vector3 lookTarget = player.position + Vector3.up * 1.5f;
        if (isAiming && aimTarget != null) lookTarget = aimTarget.position;
        transform.LookAt(lookTarget);

        // 3. Dynamic FOV
        float speed = playerRb != null ? playerRb.velocity.magnitude : 0f;
        float targetFOV = baseFOV;
        if (isSkydiving) targetFOV = Mathf.Lerp(baseFOV, maxSkydiveFOV, speed / 50f);
        else if (isAiming) targetFOV = adsFOV;
        else if (isInVehicle) targetFOV = 70f;

        cam.fieldOfView = Mathf.Lerp(cam.fieldOfView, targetFOV, Time.deltaTime * 5f);
    }

    // Call these from your Player script
    public void SetSkydiving(bool value) => isSkydiving = value;
    public void SetAiming(bool value) => isAiming = value;
    public void SetVehicle(bool value) => isInVehicle = value;
  public void SetBossNearby(bool value) => isBossNearby = value;
}
